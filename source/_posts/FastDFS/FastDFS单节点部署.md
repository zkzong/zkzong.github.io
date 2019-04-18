---
title: FastDFS单节点部署
date: 2019-04-18
categories: FastDFS
---

FastDFS是由淘宝的余庆先生所开发，是一个轻量级、高性能的开源分布式文件系统，用纯C语言开发，包括文件存储、文件同步、文件访问（上传、下载）、存取负载均衡、在线扩容、相同内容只存储一份等功能，适合有大容量存储需求的应用或系统。做分布式系统开发时，其中要解决的一个问题就是图片、音视频、文件共享的问题，分布式文件系统正好可以解决这个需求。[同类的分布式文件系统](http://elf8848.iteye.com/blog/1724382)有谷歌的GFS、HDFS（Hadoop）、TFS（淘宝）等。

源码开放下载地址：[https://github.com/happyfish100](https://github.com/happyfish100) 
早期源码开放下载地址：[https://sourceforge.net/projects/fastdfs/files/](https://sourceforge.net/projects/fastdfs/files/) 
官网论坛：[http://bbs.chinaunix.net/forum-240-1.html](http://bbs.chinaunix.net/forum-240-1.html)

## FastDFS系统架构

![FastDFS系统架构](https://upload-images.jianshu.io/upload_images/292448-bd5646140d1530be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## FastDFS文件上传流程

![FastDFS文件上传流程](https://upload-images.jianshu.io/upload_images/292448-071450869726f931.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    1. client询问tracker上传到的storage，不需要附加参数； 
    2. tracker返回一台可用的storage； 
    3. client直接和storage通讯完成文件上传。 

## FastDFS文件下载流程

![FastDFS文件下载流程](https://upload-images.jianshu.io/upload_images/292448-930ea1317ab7629c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    1. client询问tracker下载文件的storage，参数为文件标识（组名和文件名）； 
    2. tracker返回一台可用的storage； 
    3. client直接和storage通讯完成文件下载。

## 术语

FastDFS两个主要的角色：Tracker Server 和 Storage Server 
**Tracker Server：**跟踪服务器，主要负责调度storage节点与client通信，在访问上起负载均衡的作用，和记录storage节点的运行状态，是连接client和storage节点的枢纽。 
**Storage Server：**存储服务器，保存文件和文件的meta data（元数据） 
**Group：**文件组，也可以称为卷。同组内服务器上的文件是完全相同的，做集群时往往一个组会有多台服务器，上传一个文件到同组内的一台机器上后，FastDFS会将该文件即时同步到同组内的其它所有机器上，起到备份的作用。 
**meta data：**文件相关属性，键值对（Key Value Pair）方式，如：width=1024, height=768。和阿里云OSS的meta data相似。

## FastDFS单节点安装 - 服务器规划

跟踪服务器（Tracker Server）：ip01
存储服务器（Storage Server）：ip02
操作系统：CentOS7 
用户：root 
数据存储目录：/fastdfs/tracker

安装包： 
**FastDFS_v5.08.tar.gz**：FastDFS源码 
**libfastcommon-master.zip**：（从 FastDFS 和 FastDHT 中提取出来的公共 C 函数库） 
**fastdfs-nginx-module-master.zip**：storage节点http服务nginx模块 
**nginx-1.10.0.tar.gz**：Nginx安装包 
**ngx_cache_purge-2.3.tar.gz**：图片缓存清除Nginx模块（集群环境会用到） 
[点击这里](https://pan.baidu.com/s/1mVbUDICjR_SII5YTx31yRA)下载所有安装包，你也可以从[作者github](https://github.com/happyfish100)官网去下载。

**下载完成后，将压缩包解压到`/usr/local/src`目录下**

## 一、所有tracker和storage节点都执行如下操作

### 1、安装所需的依赖包
```
yum install make cmake gcc gcc-c++
```
### 2、安装libfatscommon
```
cd /usr/local/src 
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
#### 采用默认安装方式，相应的文件与目录如下：

**1、 服务脚本：**
```
/etc/init.d/fdfs_storaged 
/etc/init.d/fdfs_trackerd
```
**2、 配置文件（示例配置文件）：**
```
ll /etc/fdfs/
-rw-r--r-- 1 root root  1461 1月   4 14:34 client.conf.sample 
-rw-r--r-- 1 root root  7927 1月   4 14:34 storage.conf.sample 
-rw-r--r-- 1 root root  7200 1月   4 14:34 tracker.conf.sample
```
**3、 命令行工具（/usr/bin目录下）**
```
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

**1、 复制tracker样例配置文件，并重命名**
```
cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf
```
**2、 修改tracker配置文件**
```
vim /etc/fdfs/tracker.conf 
# 修改的内容如下：
disabled=false              # 启用配置文件
port=22122                  # tracker服务器端口（默认22122）
base_path=/fastdfs/tracker  # 存储日志和数据的根目录
```
其它参数保留默认配置， 具体配置解释可参考官方文档说明：[http://bbs.chinaunix.net/thread-1941456-1-1.html](http://bbs.chinaunix.net/thread-1941456-1-1.html)

**3、 创建base_path指定的目录**
```
mkdir -p /fastdfs/tracker
```
**4、 防火墙中打开tracker服务器端口（ 默认为 22122）**
```
vi /etc/sysconfig/iptables
```
**附加**
若`/etc/sysconfig`目录下没有iptables文件可随便写一条iptables命令配置个防火墙规则，如：

    iptables -P OUTPUT ACCEPT

然后用命令：`service iptables save`进行保存，默认就保存到 `/etc/sysconfig/iptables`文件里。这时既有了这个文件。防火墙也可以启动了。接下来要写策略，也可以直接写在`/etc/sysconfig/iptables`里了。

添加如下端口行： 
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22122 -j ACCEPT 
```
重启防火墙
```
service iptables restart
```
如果是CentOS 7.2之后的版本，可以使用如下命令：
```
# 查询端口是否开放
firewall-cmd --query-port=8080/tcp
# 开放80端口
firewall-cmd --permanent --add-port=80/tcp
# 移除端口
firewall-cmd --permanent --remove-port=8080/tcp

#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload

# 参数解释
1、firwall-cmd：是Linux提供的操作firewall的一个工具；
2、--permanent：表示设置为持久；
3、--add-port：标识添加的端口；
```

**5、 启动tracker服务器**
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

**1、 复制storage样例配置文件，并重命名**
```
cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf
```
**2、 编辑配置文件**
```
vi /etc/fdfs/storage.conf 
# 修改的内容如下:
disabled=false                      # 启用配置文件
port=23000                          # storage服务端口
base_path=/fastdfs/storage          # 数据和日志文件存储根目录
store_path0=/fastdfs/storage        # 第一个存储目录
tracker_server=ip01:22122  # tracker服务器IP和端口
http.server_port=8888               # http访问文件的端口
```
其它参数保留默认配置， 具体配置解释可参考官方文档说明：[http://bbs.chinaunix.net/thread-1941456-1-1.html](http://bbs.chinaunix.net/thread-1941456-1-1.html)

**3、 创建基础数据目录**
```
mkdir -p /fastdfs/storage
```
**4、 防火墙中打开storage服务器端口（ 默认为 23000）**
```
vi /etc/sysconfig/iptables 

#添加如下端口行： 
-A INPUT -m state --state NEW -m tcp -p tcp --dport 23000 -j ACCEPT
```
重启防火墙
```
service iptables restart
```
**5、 启动storage服务器**
```
/etc/init.d/fdfs_storaged start
```
初次启动，会在`/fastdfs/storage`目录下生成logs、data两个目录。

检查FastDFS Tracker Server是否启动成功：
```
ps -ef | grep fdfs_storaged 
```

## 四、文件上传测试（ip01）

### 1、 修改Tracker服务器客户端配置文件
```
cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf
vim /etc/fdfs/client.conf 

# 修改以下配置，其它保持默认
base_path=/fastdfs/tracker
tracker_server=ip01:22122
```
### 2、 执行文件上传命令
```
#/usr/local/src/test.png 是需要上传文件路径

/usr/bin/fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/test.png
```
返回文件ID号：`group1/M00/00/00/tlxkwlhttsGAU2ZXAAC07quU0oE095.png`
（能返回以上文件ID，说明文件已经上传成功）

如果上传报如下错误：
![上传报错](https://upload-images.jianshu.io/upload_images/292448-68d969ae3a684955.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
[root@localhost opt]# /usr/bin/fdfs_upload_file /etc/fdfs/client.conf /opt/bd.png 
[2018-07-20 16:49:05] ERROR - file: tracker_proto.c, line: 48, server: 192.168.88.61:22122, response status 2 != 0
tracker_query_storage fail, error no: 2, error info: No such file or directory
```
则**关闭storage的防火墙**。

## 五、在所有storage节点安装fastdfs-nginx-module

**1、 fastdfs-nginx-module 作用说明** 
FastDFS 通过 Tracker 服务器，将文件放在 Storage 服务器存储，但是同组存储服务器之间需要进行文件复制，有同步延迟的问题。假设 Tracker 服务器将文件上传到了 ip01，上传成功后文件 ID 已经返回给客户端。此时 FastDFS 存储集群机制会将这个文件同步到同组存储 ip02，在文件还没有复制完成的情况下，客户端如果用这个文件 ID 在 ip02 上取文件，就会出现文件无法访问的错误。而 fastdfs-nginx-module 可以重定向文件连接到源服务器取文件，避免客户端由于复制延迟导致的文件无法访问错误。(解压后的 fastdfs-nginx-module 在 nginx 安装时使用)

**2、 解压fastdfs-nginx-module-master.zip**
```
cd /usr/local/src
unzip fastdfs-nginx-module-master.zip
```
**3、 修改 fastdfs-nginx-module 的 config 配置文件**
```
cd fastdfs-nginx-module/src
vi config
```
将
```
CORE_INCS="$CORE_INCS /usr/local/include/fastdfs /usr/local/include/fastcommon/"
CORE_LIBS="$CORE_LIBS -L/usr/local/lib -lfastcommon -lfdfsclient"
```
修改为：
```
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
CORE_LIBS="$CORE_LIBS -L/usr/lib -lfastcommon -lfdfsclient"
```
**4、上传当前的稳定版本 Nginx(nginx-1.6.2.tar.gz)到/usr/local/src 目录**

**5、安装编译 Nginx 所需的依赖包**
```
yum install gcc gcc-c++ make automake autoconf libtool pcre* zlib openssl openssl-devel
```

**6、编译安装 Nginx (添加 fastdfs-nginx-module 模块)**
```
cd /usr/local/src/
tar -zxvf nginx-1.6.2.tar.gz
cd nginx-1.6.2
./configure --add-module=/usr/local/src/fastdfs-nginx-module/src
make && make install
```

**7、复制 fastdfs-nginx-module 源码中的配置文件到/etc/fdfs 目录,并修改**
```
cp /usr/local/src/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/ 
vi /etc/fdfs/mod_fastdfs.conf
```

**8、修改以下配置**
```
connect_timeout=10     
base_path=/tmp    
tracker_server=ip01:22122     
storage_server_port=23000     
group_name=group1     
url_have_group_name = true     
store_path0=/fastdfs/storage
```

**9、复制 FastDFS 的部分配置文件到/etc/fdfs 目录**
```
cd /usr/local/src/FastDFS/conf
cp http.conf mime.types /etc/fdfs/
```

**10、在`/fastdfs/storage`文件存储目录下创建软连接,将其链接到实际存放数据的目录**
```
ln -s /fastdfs/storage/data/ /fastdfs/storage/data/M00
```

**11、配置 Nginx（/usr/local/nginx/conf/nginx.conf）**

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
 
    server {
        listen       8888;
        server_name  localhost;
 
        #charset koi8-r;
 
        #access_log  logs/host.access.log  main;
 
        location / {
            root   html;
            index  index.html index.htm;
        }
        location ~/group([0-9])/M00 {
            alias /fastdfs/storage;
            ngx_fastdfs_module;
        }
        #error_page  404              /404.html;
 
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
 
        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}
 
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}
 
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
 
 
    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;
 
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
 
 
    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;
 
    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;
 
    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;
 
    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;
 
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
}
```

 A、`8888`端口值是要与`/etc/fdfs/storage.conf`中的 `http.server_port=8888`相对应，因为`http.server_port`默认为`8888`，如果想改成 80，则要对应修改过来。
 B、Storage 对应有多个 group 的情况下，访问路径带 group 名，如`/group1/M00/00/00/xxx`，对应的 Nginx 配置为：
```
location ~/group([0-9])/M00 {
         ngx_fastdfs_module;
}
```
C、如查下载时如发现老报 404，将 nginx.conf 第一行 user nobody 修改为 user root 后重新启动。

**12、防火墙中打开 Nginx 的 8888 端口**
```
vi /etc/sysconfig/iptables
```

 添加：
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8888 -j ACCEPT 

#重启防火墙
service iptables restart
```

启动 Nginx
```
 /usr/local/nginx/sbin/nginx
(重启 Nginx 的命令为:/usr/local/nginx/sbin/nginx -s reload)

# 或者
cd /usr/local/nginx/sbin/
./nginx -s reload
```

 **13、通过浏览器访问测试时上传的文件**
```
http://ip:8888/group1/M00/00/00/tlxkwlhttsGAU2ZXAAC07quU0oE095.png
```

参考文章：
https://www.cnblogs.com/cnmenglang/p/6251696.html
https://www.cnblogs.com/chiangchou/p/fastdfs.html
