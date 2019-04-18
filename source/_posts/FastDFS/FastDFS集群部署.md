---
title: FastDFS集群部署
date: 2019-04-18
categories: FastDFS
---

之前介绍过关于FastDFS单机部署，详见博文：[FastDFS单节点部署](https://www.jianshu.com/p/a4fcba0ece9a)

下面来看下FastDFS集群部署，实现高可用（HA）。

## 服务器规划
**跟踪服务器**
跟踪服务器1【主机】（Tracker Server）：192.100.139.121
跟踪服务器2【备机】（Tracker Server）：192.100.139.122
**group1存储服务器**
存储服务器1（Storage Server）：192.100.139.123
存储服务器2（Storage Server）：192.100.139.124
**group2存储服务器**
存储服务器3（Storage Server）：192.100.139.125
存储服务器4（Storage Server）：192.100.139.126

操作系统：CentOS7 
用户：root 

数据存储目录：/fastdfs/storage

## 安装包： 

1.  FastDFS_v5.08.tar.gz：FastDFS源码 
2.  libfastcommon-master.zip：（从 FastDFS 和 FastDHT 中提取出来的公共 C 函数库） 
3.  fastdfs-nginx-module-master.zip：storage节点http服务nginx模块 
4.  nginx-1.10.0.tar.gz：Nginx安装包 
5.  ngx_cache_purge-2.3.tar.gz：图片缓存清除Nginx模块（集群环境会用到） 

[点击这里](https://pan.baidu.com/s/1mVbUDICjR_SII5YTx31yRA)下载所有安装包。

下载完成后，将压缩包解压到/usr/local/src目录下。

## 一、所有tracker和storage节点都执行如下操作

### 1、安装所需的依赖包
```
yum install make cmake gcc gcc-c++
```

### 2、安装libfatscommon
```
cd /usr/local/src
#安装unzip 命令： yum install -y unzip zip
unzip libfastcommon-master.zip
cd libfastcommon-master
## 编译、安装
./make.sh
./make.sh install
```

### 3、安装FastDFS
```
cd /usr/local/src
tar -xzvf FastDFS_v5.08.tar.gz
cd FastDFS
./make.sh
./make.sh install
```
 **采用默认安装方式，相应的文件与目录检查如下：**

**1、服务脚本**
```
/etc/init.d/fdfs_storaged
/etc/init.d/fdfs_trackerd
```
**2、配置文件（示例配置文件）**
```
ll /etc/fdfs/
-rw-r--r-- 1 root root  1461 1月   4 14:34 client.conf.sample
-rw-r--r-- 1 root root  7927 1月   4 14:34 storage.conf.sample
-rw-r--r-- 1 root root  7200 1月   4 14:34 tracker.conf.sample
```
 **3、命令行工具（`/usr/bin`目录下）**
```
ll /usr/bin/fdfs_*
-rwxr-xr-x    1 root root     260584 1月   4 14:34 fdfs_appender_test
-rwxr-xr-x    1 root root     260281 1月   4 14:34 fdfs_appender_test1
-rwxr-xr-x    1 root root     250625 1月   4 14:34 fdfs_append_file
-rwxr-xr-x    1 root root     250045 1月   4 14:34 fdfs_crc32
-rwxr-xr-x    1 root root     250708 1月   4 14:34 fdfs_delete_file
-rwxr-xr-x    1 root root     251515 1月   4 14:34 fdfs_download_file
-rwxr-xr-x    1 root root     251273 1月   4 14:34 fdfs_file_info
-rwxr-xr-x    1 root root     266401 1月   4 14:34 fdfs_monitor
-rwxr-xr-x    1 root root     873233 1月   4 14:34 fdfs_storaged
-rwxr-xr-x    1 root root     266952 1月   4 14:34 fdfs_test
-rwxr-xr-x    1 root root     266153 1月   4 14:34 fdfs_test1
-rwxr-xr-x    1 root root     371336 1月   4 14:34 fdfs_trackerd
-rwxr-xr-x    1 root root     251651 1月   4 14:34 fdfs_upload_appender
-rwxr-xr-x    1 root root     252781 1月   4 14:34 fdfs_upload_file
```

## 二、配置tracker服务器

### 1、复制tracker样例配置文件，并重命名
```
cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf
```

###  2、修改tracker配置文件
```
vim /etc/fdfs/tracker.conf
# 修改的内容如下：
disabled=false # 启用配置文件
port=22122 # tracker服务器端口（默认22122）
base_path=/fastdfs/tracker  # 存储日志和数据的根目录
store_lookup=0 # 轮询方式上传
```

其它参数保留默认配置， 具体配置解释可参考官方文档说明：http://bbs.chinaunix.net/thread-1941456-1-1.html

### 3、创建base_path指定的目录
```
mkdir -p /fastdfs/tracker
```

### 4、防火墙中打开tracker服务器端口（ 默认为 22122）
```
vi /etc/sysconfig/iptables
```
附加：若`/etc/sysconfig`目录下没有iptables文件可随便写一条iptables命令配置个防火墙规则，如：
```
iptables -P OUTPUT ACCEPT
```
然后用命令：`service iptables save`进行保存，默认就保存到 `/etc/sysconfig/iptables`文件里。这时既有了这个文件，防火墙也可以启动了。接下来要写策略，也可以直接写在`/etc/sysconfig/iptables`里了。
添加如下端口行：
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22122 -j ACCEPT
```
重启防火墙
```
service iptables restart
```

### 5、启动tracker服务器
```
/etc/init.d/fdfs_trackerd start
```

初次启动，会在/fastdfs/tracker目录下生成logs、data两个目录。
```
drwxr-xr-x 2 root root 4096 1月   4 15:00 data
drwxr-xr-x 2 root root 4096 1月   4 14:38 logs
```
检查FastDFS Tracker Server是否启动成功：
```
ps -ef | grep fdfs_trackerd
```
## 三、配置storage服务器

### 1、复制storage样例配置文件，并重命名
```
cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf
```
### 2、编辑配置文件
```
vi /etc/fdfs/storage.conf
# 修改的内容如下:
disabled=false # 启用配置文件
port=23000 # storage服务端口
base_path=/fastdfs/storage          # 数据和日志文件存储根目录
store_path0=/fastdfs/storage        # 第一个存储目录
tracker_server=192.100.139.121:22122 # tracker服务器IP和端口
tracker_server=192.100.139.122:22122 #tracker服务器IP2和端口
http.server_port=8888 # http访问文件的端口
```

**配置group_name**
不同分组配置不同group_name
```
group_name=group1
group_name=group2
```

其它参数保留默认配置， 具体配置解释可参考官方文档说明：http://bbs.chinaunix.net/thread-1941456-1-1.html

### 3、创建基础数据目录
```
mkdir -p /fastdfs/storage
```

### 4、防火墙中打开storage服务器端口（ 默认为 23000）
```
vi /etc/sysconfig/iptables
```
添加如下端口行：
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 23000 -j ACCEPT
```
重启防火墙
```
service iptables restart
```

**可以通过`/fastdfs/storage/logs/storaged.log`查看启动日志。**

### 5、启动storage服务器
```
/etc/init.d/fdfs_storaged start
```
初次启动，会在/fastdfs/storage目录下生成logs、data两个目录。
```
drwxr-xr-x 259 root root 4096 Mar 31 06:22 data
drwxr-xr-x   2 root root 4096 Mar 31 06:22 logs
```
检查FastDFS Tracker Server是否启动成功：
```
[root@gyl-test-t9 ~]# ps -ef | grep fdfs_storaged
root 1336     1  3 06:22 ?        00:00:01 /usr/bin/fdfs_storaged /etc/fdfs/storage.conf
root 1347   369  0 06:23 pts/0    00:00:00 grep fdfs_storaged
```
## 四、文件上传测试（ip01）

### 1、修改Tracker服务器客户端配置文件
```
cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf
vim /etc/fdfs/client.conf
# 修改以下配置，其它保持默认
base_path=/fastdfs/tracker
tracker_server=192.100.139.121:22122 # tracker服务器IP和端口
tracker_server=192.100.139.122:22122  #tracker服务器IP2和端口
```

### 2、执行文件上传命令
```
#/usr/local/src/test.png 是需要上传文件路径 
/usr/bin/fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/test.png
返回文件ID号：group1/M00/00/00/tlxkwlhttsGAU2ZXAAC07quU0oE095.png
（能返回以上文件ID，说明文件已经上传成功）
```
或者
```
/usr/bin/fdfs_test /etc/fdfs/client.conf upload client.conf
```
# ![image](http://upload-images.jianshu.io/upload_images/292448-caf5fea892ba4587.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 五、在所有storage节点安装fastdfs-nginx-module

### 1、fastdfs-nginx-module 作用说明

**FastDFS通过Tracker服务器，将文件放在Storage服务器存储，但是同组存储服务器之间需要进入文件复制，有同步延迟的问题。假设Tracker服务器将文件上传到了ip01，上传成功后文件ID已经返回给客户端。此时FastDFS存储集群机制会将这个文件同步到同组存储ip02，在文件还没有复制完成的情况下，客户端如果用这个文件ID在ip02上取文件，就会出现文件无法访问的错误。而fastdfs-nginx-module可以重定向文件连接到源服务器取文件，避免客户端由于复制延迟导致的文件无法访问错误。(解压后的fastdfs-nginx-module在nginx安装时使用)**

### 2、解压 fastdfs-nginx-module_v1.16.tar.gz
```
cd /usr/local/src
tar -xzvf fastdfs-nginx-module_v1.16.tar.gz
```

### 3、修改 fastdfs-nginx-module 的 config 配置文件 
```
cd fastdfs-nginx-module/src
vim config
```
将
```
CORE_INCS="$CORE_INCS /usr/local/include/fastdfs /usr/local/include/fastcommon/"
```
修改为：
```
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
```

### 4、安装编译 Nginx 所需的依赖包
```
yum install gcc gcc-c++ make automake autoconf libtool pcre* zlib openssl openssl-devel
```

### 5、上传当前的稳定版本 Nginx(nginx-1.6.2.tar.gz)到 /usr/local/src 目录

### 6、编译安装 Nginx (添加 fastdfs-nginx-module 模块) 
```
cd /usr/local/src/
tar -zxvf nginx-1.10.0.tar.gz
cd nginx-1.10.0
./configure --prefix=/opt/nginx --add-module=/usr/local/src/fastdfs-nginx-module/src
make && make install
```

### 7、复制 fastdfs-nginx-module 源码中的配置文件到 /etc/fdfs 目录，并修改
```
cp /usr/local/src/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/
vi /etc/fdfs/mod_fastdfs.conf
```
修改以下配置：
```
connect_timeout=10
base_path=/tmp
tracker_server=192.100.139.121:22122 # tracker服务器IP和端口
tracker_server=192.100.139.122:22122 #tracker服务器IP2和端口
group_name=group1                 #当前服务器的group名
url_have_group_name=true #url中包含group名称
store_path0=/fastdfs/storage #存储路径
group_count=2                   #设置组的个数

#在最后添加 
[group1]
group_name=group1
storage_server_port=23000
store_path_count=1
store_path0=/fastdfs/storage

[group2]
group_name=group2
storage_server_port=23000
store_path_count=1
store_path0=/fastdfs/storage
```

**注意group_name的设置，如果是group1就设置为group1，如果是group2就设置为group2**

### 8、复制 FastDFS 的部分配置文件到 /etc/fdfs 目录
```
cd /usr/local/src/FastDFS/conf
cp http.conf mime.types /etc/fdfs/
```

### 9、在 /fastdfs/storage 文件存储目录下创建软连接，将其链接到实际存放数据的目录
```
ln -s /fastdfs/storage/data/ /fastdfs/storage/data/M00
```

### 10、配置 Nginx（/usr/local/nginx/conf/nginx.conf）
```
user nobody;
worker_processes 1;
events {
    worker_connections 1024;
}
http {
    include mime.types;
    default_type application/octet-stream;
    sendfile on;
    keepalive_timeout 65;
    server {
        listen 8888;
        server_name localhost;
        location ~/group[1-2]/M00 {
            ngx_fastdfs_module;
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root html;
        }
    }

}
```

**说明：** 

 A、8888 端口值是要与`/etc/fdfs/storage.conf`中的 `http.server_port=8888` 相对应，因为 `http.server_port` 默认为 8888，如果想改成 80，则要对应修改过来。
 B、Storage 对应有多个 group 的情况下，访问路径带 group 名，如/group1/M00/00/00/xxx，对应的 Nginx 配置为：
```
location ~/group([0-9])/M00 {
       ngx_fastdfs_module;
}
```
C、如查下载时如发现老报 404,将 nginx.conf 第一行 `user nobody` 修改为 `user root` 后重新启动。

### 11、防火墙中打开 Nginx 的 8888 端口
```
vi /etc/sysconfig/iptables
```
添加：
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8888 -j ACCEPT
```
重启防火墙
```
service iptables restart
```
启动nginx：
```
/usr/local/nginx/sbin/nginx
(重启 Nginx 的命令为:/usr/local/nginx/sbin/nginx -s reload)
```

## 六、验证：通过浏览器访问测试时上传的文件

切换追踪服务器IP同样可以访问

http://192.100.139.123:8888/group1/M00/00/00/CmSKtFj13gyAen4oAAH0yXi-HW8296.png

http://192.100.139.124:8888/group1/M00/00/00/CmSKtFj13gyAen4oAAH0yXi-HW8296.png

**因为缓存问题，125、126两台机器也能通过url访问group1上的文件**

## 七、tracker节点安装nginx

### 1、下载需要的依赖库文件
```
yum -y install pcre pcre-devel zlib zlib-devel
```

### 2、解压并安装nginx，加入ngx_cache_purge（加入缓存模块）
```
tar -zxvf nginx-1.10.0.tar.gz
tar -zxvf ngx_cache_purge-2.3.tar.gz

cd nginx-1.10.0
./configure --add-module=/usr/local/src/ngx_cache_purge-2.3
make && make install
```

### 3、配置nginx负载均衡和缓存
```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    #设置group1的服务器
    upstream fdfs_group1 {
        server 192.100.139.123:8888 weight=1 max_fails=2 fail_timeout=30s;
        server 192.100.139.124:8888 weight=1 max_fails=2 fail_timeout=30s;
    }
    #设置group2的服务器
    upstream fdfs_group2 {
        server 192.100.139.125:8888 weight=1 max_fails=2 fail_timeout=30s;
        server 192.100.139.126:8888 weight=1 max_fails=2 fail_timeout=30s;
    }

    server {
        listen       8000;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #设置group1的负载均衡参数
        location /group1/M00 {
            proxy_pass http://fdfs_group1;
        }
        #设置group2的负载均衡参数
        location /group2/M00 {
            proxy_pass http://fdfs_group2;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }

}
```

### 4、防火墙设置

参考上面设置

### 5、启动nginx
```
/usr/local/nginx/sbin/nginx
```

### 6、上传文件
```
/usr/bin/fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/test.png
```

### 7、通过tracker地址访问
http://192.100.139.121:8000/group2/M00/00/00/wKhYdFtdrv-AdFphAAAexdAvMYY481.png

http://192.100.139.121:8000/group2/M00/00/00/wKhYdFtdrv-AdFphAAAexdAvMYY481.png

## 八、Java API 客户端配置

### 1、前往GitHub下载Java_client代码。[https://github.com/fzmeng/fastdfs.client](https://github.com/fzmeng/fastdfs.client)

### 2.在你的项目src/java/resources 下加入文件 fastdfs_client.conf

注意修改tracker服务器Ip地址
```
connect_timeout = 2
network_timeout = 30
charset = ISO8859-1
http.tracker_http_port = 8888
http.anti_steal_token = no
tracker_server=192.100.139.121:22122
tracker_server=192.100.139.122:22122
default_group_name=group1
```
