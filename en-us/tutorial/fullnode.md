## Setup public chain full node

Unlike observation nodes and production nodes, full nodes provide more comprehensive data storage and statistics functions.
This tutorial will show you how to setup a Contentos public chain full node step by step.

### 1.Install MySQL

Part of the data storage and statistics functions of the full node depends on MySQL,
so you need to install MySQL before everything started. We recommend MySQL version above [8.0](https://dev.mysql.com/downloads/mysql/)

### 2.Install Go environment

The main development language of the Contentos project is golang, and the go modules officially provided by golang are used for dependency management, so at least [Go 1.11](https://golang.org/dl/) or higher is required.
Since [Go 1.12](https://golang.org/dl/) updates the modules mechanism, we recommend Go 1.12 and later versions.
Currently the most standard version is Go 1.11.4.

### 3.Pull code, compile, initialize configuration file, modify configuration information

First, you need to execute the following instructions in order to pull the code to the local and compile the cosd executable

```
git clone git@github.com:coschain/contentos-go.git
cd contentos-go/cmd/cosd/
go build
```

Next, you need to execute the following initialization instruction to initialize the configuration file required for the full node operation

```
./cosd init
```

After the configuration file is initialized, a file named `config.toml` will be generated in the `~/.coschain/cosd` folder of your machine, which provides the configuration information required for the node to run. 
At this point, you need to modify the following in the file

```
  BootStrap = false (Be careful, this must be set to false)
  LocalBpName = ""
  LocalBpPrivateKey = ""

  [Database]
    Db = "contentosdb"
    Driver = "mysql"
    Password = your_mysql_password
    User = your_mysql_account_name (Account name created when installing MySQL. If not, the default is root.)

  ...

  eedList = ["3.210.182.21:20338","34.206.144.13:20338"] (Set this to the seed nodes of contentos main net)
```

### 4.Initialize the MySQL environment

First, you need to log in to MySQL and manually create a database named `contentosdb`

Then, execute the following instruction to initialize the various tables needed in the database
```
./cosd db init
```

### 5.Run full node

At this point, you can start your full node with the following instruction
```
./cosd start
```

when starting the node, you can specify the type of plugin you want to launch.
For more information about plugins, you can refer to this article [Plugins](https://github.com/coschain/cos-document/blob/master/en-us/plugins.md),
different plugins correspond to different statistics.

For example, I want to start these four plugins `trxsqlservice`,` dailystatservice`, `block_log_svc`,` block_log_proc_svc`, then my startup command will be like
```
./cosd start --plugin=trxsqlservice --plugin=dailystatservice --plugin=block_log_svc --plugin=block_log_proc_svc
```