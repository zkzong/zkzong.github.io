随着github的普及和流行，现在程序员可能都习惯把代码托管到类似github的远程仓库中。毫无疑问，github是最受欢迎的托管平台。但是由于网络等种种原因，github在国内的访问并不稳定。于是国内各种托管平台应运而生，比较知名的有开源中国、coding等。很多国内程序员会把代码托管到多个平台，兼顾稳定性和流行性。

那么如何方便快捷的把代码托管到多个平台呢？

例如我有下面两个仓库：
`https://gitee.com/zkzong/mongodb.git`
`https://github.com/zkzong/mongodb.git`

先添加第一个仓库：
```
git remote add origin https://gitee.com/zkzong/mongodb.git
```
再添加第二个仓库：
```
git remote set-url --add origin https://github.com/zkzong/mongodb.git
```

如果还有其他，则可以像添加第二个一样继续添加其他仓库。

然后使用下面命令提交：
```
git push origin --all
```
打开`.git/config`，可以看到这样的配置
```
[remote "origin"]
	url = https://github.com/zkzong/spring-boot.git
	fetch = +refs/heads/*:refs/remotes/origin/*
	url = https://gitee.com/zkzong/spring-boot.git
```

刚才的命令其实就是添加了这些配置。如果不想用命令行，可以直接编辑该文件，添加对应的url即可。
