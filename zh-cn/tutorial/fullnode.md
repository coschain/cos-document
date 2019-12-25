## 搭建公链全节点

与观察节点和生产节点不同，全节点提供了更为全面的数据存储与统计功能，本教程将一步步的向您展示如何搭建一个Contentos公链全节点

### 1.安装 MySQL

全节点的部分数据存储与统计功能需要依赖MySQL，所以在一切工作开始之前，您需要先安装MySQL。我们建议MySQL的版本在 [8.0](https://dev.mysql.com/downloads/mysql/) 以上

### 2.安装Go环境

Contentos 项目主要开发语言是 golang，并且使用了 golang 官方提供的 go modules 作为依赖管理的工具，所以至少需要 [Go 1.11](https://golang.org/dl/) 以上的版本。由于 [Go 1.12](https://golang.org/dl/) 对 modules 机制的更新，所以更推荐 Go 1.12 以后的版本。
目前最标准的版本是 Go 1.11.4。

### 3.拉取代码、编译、初始化配置文件、修改配置信息

首先，您需要依次执行以下指令将代码拉取到本地并编译cosd可执行文件

```
git clone git@github.com:coschain/contentos-go.git
cd contentos-go/cmd/cosd/
go build
```

接着，您需要执行如下的初始化指令，初始化全节点运行需要的配置文件

```
./cosd init
```

配置文件初始化完成之后，在您机器的`~/.coschain/cosd`文件夹下，会生成一个名为`config.toml`的配置文件，该文件提供了节点运行需要的配置信息。
此时，您需要对文件中的如下内容进行修改

```
  BootStrap = false (注意，这个值必须设置为false)
  LocalBpName = ""
  LocalBpPrivateKey = ""

  [Database]
    Db = "contentosdb"
    Driver = "mysql"
    Password = your_mysql_password
    User = your_mysql_account_name(安装MySQL时创建的账号名，若没有创建，默认是root)

  ...

  eedList = ["3.210.182.21:20338","34.206.144.13:20338"] (主网的种子节点，需要正确配置才能保证连上网络中其他节点)
```

### 4.初始化MySQL环境

首先，您需要登录MySQL，手动创建一个名为`contentosdb`的数据库

然后，执行以下代码，初始化数据库中需要的各种 table
```
./cosd db init
```

### 5.运行全节点

至此，您可以通过如下指令启动您的全节点

```
./cosd start
```

在启动的时候，您可以指定您想要启动的插件类型。关于插件的详细信息，您可以参考这篇文章[插件](https://github.com/coschain/cos-document/blob/master/zh-cn/plugins.md)，不同的插件，对应了不同的统计信息

例如，我要启动`trxsqlservice`、`dailystatservice`、`block_log_svc`、`block_log_proc_svc`四个插件，那么我的启动命令就是这样的

```
./cosd start --plugin=trxsqlservice --plugin=dailystatservice --plugin=block_log_svc --plugin=block_log_proc_svc
```