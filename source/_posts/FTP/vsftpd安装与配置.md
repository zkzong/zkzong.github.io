---
title: vsftpd安装与配置
date: 2019-04-19
categories: FTP
---

## 企业环境
公司为了宣传最新的产品信息，计划搭建 FTP 服务器，为客户提供相关文档的下载。对所有互联网开放共享目录，允许下载产品信息，禁止上传。公司的合作单位能够使用 FTP 服务器进行上传和下载，但不可以删除数据。并且保证服务器的稳定性，进行适当优化设置。
## 需求分析
根据企业的需求，对于不同用户进行不同的权限限制，FTP 服务器需要实现用户的审核。需考虑到服务器的安全性，所以关闭实体用户登录，使用虚拟帐号验证机制，并对不同虚拟帐号设置不同的权限。为了保证服务器的性能，还需要根据用户的等级，限制客户端的连接数及下载速度。

## 安装
```
yum -y install vsftpd
```

## 解决方案
### 1、创建用户数据库
（1）创建用户文本文件
先建立用户文本文件 `vsftpd_virtualuser.txt`，添加两个虚拟帐号，公共帐号 ftp 及客户帐号 vip。
```
touch /etc/vsftpd/vsftpd_virtualuser.txt
vim /etc/vsftpd/vsftpd_virtualuser.txt
```
格式：
```
虚拟帐号 1
密码
虚拟帐号 2
密码
```
本文使用账号和密码：
```
ftp
123456
vip
456789
```
保存退出。
（2）生成数据库
保存虚拟帐号和密码的文本文件无法被系统帐号直接调用。我们需要使用 `db_load` 命令生成 db 数据库文件。
```
db_load -T -t hash -f /etc/vsftpd/vsftpd_virtualuser.txt /etc/vsftpd/vsftpd_virtualuser.db
```
（3）修改数据库文件访问权限
数据库文件中保存着虚拟帐号的密码信息，为了防止非法用户盗取，我们可以修改该文件的访问权限。生成的认证文件的权限应设置为只对 root 用户可读可写，即 600。
```
chmod 600 /etc/vsftpd/vsftpd_virtualuser.db
```
### 2、配置 PAM 文件
为了使服务器能够使用数据库文件，对客户端进行身份验证，需要调用系统的 PAM 模块.PAM(Plugable Authentication Module)为可插拔认证模块，不必重新安装应用系统，通过修改指定的配置文件，调整对该程序的认证方式。PAM 模块配置文件路径为/etc/pam.d/目录，此目录下保存着大量与认证有关的配置文件，并以服务名称命名。
修改 vsftpd 对应的 PAM 配置文件`/etc/pam.d/vsftpd`，将默认配置使用“#”全部注释，添加相应字段。
```
auth       required     pam_userdb.so   db=/etc/vsftpd/vsftpd_virtualuser
account    required     pam_userdb.so   db=/etc/vsftpd/vsftpd_virtualuser
```
### 3、创建虚拟帐号对应的系统用户
对于公共帐号和客户帐号，因为需要配置不同的权限，所以可以将两个帐号的目录进行隔离，控制用户的文件访问。公共帐号 ftp 对应系统帐号 ftpuser，并指定其主目录为/var/ftp/share，而客户帐号 vip 对应系统帐号 ftpvip，指定主目录为/var/ftp/vip。

如果不设置可执行用户登录会报不能更改目录错误。
```
useradd -d /var/ftp/share ftpuser
useradd -d /var/ftp/vip ftpvip

chmod -R 500 /var/ftp/share/
chmod -R 700 /var/ftp/vip/
```

公共帐号 ftp 只允许下载，修改 share 目录其他用户权限为 `rx` 可读可执行。
```
chmod -R 500 /var/ftp/share/
```
客户帐号 vip 允许上传和下载，所以对 vip 目录权限设置为 rwx，可读可写可执行。
```
chmod -R 700 /var/ftp/vip/
```

### 4、建立配置文件
设置多个虚拟帐号的不同权限，若使用一个配置文件无法实现此功能，需要为每个虚拟帐号建立独立的配置文件，并根据需要进行相应的设置。
（1）修改 vsftpd.conf 主配置文件
配置主配置文件 `/etc/vsftpd/vsftpd.conf` 添加虚拟帐号的共同设置并添加 `user_config_dir` 字段，定义虚拟帐号的配置文件目录。
禁用匿名用户登录并启用本地用户登录设置：
```
anonymous_enable=NO
local_enable=YES
```

```
# 将所有本地用户限制在家目录中，NO 则不限制
chroot_local_user=YES
# 配置 vsftpd 使用的 PAM 模块为 vsftpd
pam_service_name=vsftpd
# 设置虚拟帐号的主目录为/vuserconfig
user_config_dir=/etc/vsftpd/vuserconfig
# 设置 FTP 服务器最大接入客户端数为 300 个
max_clients=300
# 设置每个 IP 地址最大连接数为 10 个
max_per_ip=10
```
（2）建立虚拟帐号配置文件
在 `user_config_dir` 指定路径下，建立与虚拟帐号同名的配置文件并添加相应的配置字段。
首先建立公共帐号 ftp 的配置文件：
```
vim /etc/vsftpd/vuserconfig/ftp
```

```
# 开启虚拟帐号登录
guest_enable=yes
# 设置 ftp 对应的系统帐号为 ftpuser
guest_username=ftpuser
# 允许匿名用户浏览器整个服务器的文件系统
anon_world_readable_only=no
# 限定传输速率为 50KB/s
anon_max_rate=50000
```
> 注意：
vsftpd 对于文件传输速度限制并不是绝对锁定在一个数值上哈，而是在  80%~120%之间变化。比如设置 100KB/s ，则实际是速度在 80KB/s~120KB/s 之间变化。

下面是客户帐号的配置文件 vip：
```
vim /etc/vsftpd/vuserconfig/vip
```
```
# 开启虚拟帐号登录
guest_enable=yes
# 设置 ftp 对应的系统帐号为 ftpvip
guest_username=ftpvip
# 允许匿名用户浏览器整个服务器的文件系统
anon_world_readable_only=no
# 允许在文件系统写入权限
write_enable=yes
# 允许创建文件夹
anon_mkdir_write_enable=yes
# 开启匿名帐号的上传功能
anon_upload_enable=yes
# 限定传输速度为 100KB/s
anon_max_rate=100000
```
> 如果需要删除权限，可在配置中添加：
`anon_other_write_enable=YES`

### 5、重启 vsftpd 使配置生效
```
systemctl restart vsftpd
```
### 6、测试
（1）公共帐号 ftp 测试
在公共帐号测试前，我们先建立个测试文件`productinfo.xls`。
公共帐号登录 ftp 服务器
![登录](https://upload-images.jianshu.io/upload_images/292448-5d35e6e4a91285fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
登录成功
![登录成功](https://upload-images.jianshu.io/upload_images/292448-4e4d03754f95841c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
测试下载，ok，成功
![下载](https://upload-images.jianshu.io/upload_images/292448-1cb8f912ea8bbafc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
测试上传文件及文件夹，不成功
![上传失败](https://upload-images.jianshu.io/upload_images/292448-cb31165e8a41aa1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最后测试限速 50KB/s
![限速](https://upload-images.jianshu.io/upload_images/292448-dc99eae704aa45c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
达成目标。
（2）客户帐号 vip 测试
客户帐号 vip 登录
![登录](https://upload-images.jianshu.io/upload_images/292448-ac204c9cde142045.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
登录成功
![登录成功](https://upload-images.jianshu.io/upload_images/292448-d9288210fa461d98.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
测试上传，ok，成功
![上传](https://upload-images.jianshu.io/upload_images/292448-234dd604a8c0714e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
测试下载，ok，成功
![下载](https://upload-images.jianshu.io/upload_images/292448-844afa1fa769454c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
测试删除，ok，不成功
![删除](https://upload-images.jianshu.io/upload_images/292448-425fd09a48b321d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
测试限速下载 100KB/s
![限速](https://upload-images.jianshu.io/upload_images/292448-000ff79c257b347e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
达成目标需求。

**常见错误及解决办法**
1. cannot change directory
关闭SeLinux：
```
setenforce 0
```
2. 500 OOPS: vsftpd: refusing to run with writable root inside chroot()
vsftpd.conf添加配置：
```
allow_writeable_chroot=YES
```
