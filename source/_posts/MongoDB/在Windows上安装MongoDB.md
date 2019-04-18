MongoDB的下载和安装与其他软件没有什么区别，在此不再详细介绍，可以直接去官网下载安装。

作为个人学习使用，建议安装**社区版**。

下面主要介绍MongoDB的配置和运行。

## 1. 运行MongoDB

### 第一步 安装MongoDB环境

MongoDB需要data目录存储数据。它默认的data目录是`/data/db`。

可以使用`--dbpath`指定data目录，如：
```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --dbpath d:\test\mongodb\data
```
如果路径中有空格，可以使用双引号包含路径，如：
```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --dbpath "d:\test\mongo db data"
```
当然，也可以在配置文件中指定`dbpath`。

### 第二步 启动MongoDB

运行`mongod.exe`启动MongoDB。其命令如下：
```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe"
```

### 第三步 连接到MongoDB

使用`mongo.exe`连接MongoDB，其命令如下：
```
"C:\Program Files\MongoDB\Server\3.4\bin\mongo.exe
```

停止MongoDB服务，只需在运行`mongod`的命令行界面按`Ctrl+C`。

## 2. 把MongoDB配置成Windows服务

### 第一步 打开管理员命令行

打开命令行界面，然后按`Ctrl+Shift+Enter`以管理员身份运行命令行。
然后执行剩下的步骤。

### 第二步 创建目录

创建目录存放db和log文件
```
mkdir c:\data\db
mkdir c:\data\log
```

###第三步 创建配置文件

创建配置文件，该文件中必须设置`systemLog.path`，其他为可选。
例如，创建配置文件`C:\Program Files\MongoDB\Server\3.4\mongod.cfg`，在其中配置`systemLog.path`和`storage.dbPath`。
```
systemLog:
    destination: file
    path: c:\data\log\mongod.log
storage:
    dbPath: c:\data\db
```

### 第四步 安装MongoDB服务

**注意：必须以管理员权限运行下面的命令行。**

运行`mongod.exe`，并使用两个参数`--install`和`--config`，其中`--config`指定了之前创建的配置文件。

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --config "C:\Program Files\MongoDB\Server\3.4\mongod.cfg" --install
```

### 第五步 运行MongoDB服务

```
net start MongoDB
```

### 第六步 停止MongoDB服务

```
net stop MongoDB
```

移除MongoDB服务：
```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --remove
```
