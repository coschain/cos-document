# 搭建本地环境并运行

教程将会从源代码编译开始，一步步搭建一个可运行的环境，直到本地链可以正常运行并出块。考虑到可能出现的编译方面的错误，我们也计划提供了一个能正常运行本地链的 docker。目前以及为了更深入理解本地链的编译和运行，我们强烈建议从源码开始编译。

## 第一步: 获取源代码

从 github 上获取 master 分支的代码

```shell
git clone git@github.com:coschain/contentos-go.git
```

## 第二步: 编译

### Go 的版本

Contentos 项目主要开发语言是 golang，并且使用了 golang 官方提供的 go modules 作为依赖管理的工具，所以至少需要 [Go 1.11](https://golang.org/dl/) 以上的版本。由于 [Go 1.12](https://golang.org/dl/) 对 modules 机制的更新，所以更推荐 Go 1.12 以后的版本。
目前最标准的版本是 Go 1.11.4。

### 编译 cosd 和 wallet

Contentos-go 可以编译出的可执行文件都在 cmd 文件夹下。其中最重要的 cosd 以及 wallet-cli 。前者是 contentos 主链，后者是提供的与主链交互的命令行工具。

```shell
cd build
go build ../cmd/cosd
go build ../cmd/wallet-cli
```

当然你也可以通过 `go build -o` 指定名字以及位置。

## 第三步: 运行

### 初始化

contentos 需要初始化 `~/.coschain` 文件夹用于存放配置文件以及数据，以下命令将会创建默认文件夹 `~/.coschain/cosd`。

```shell
cosd init
```

也可以指定文件夹名字，以下命令将会创建文件夹 `~/.coschain/testnode`。

```shell
cosd init -n testnode
```

指定配置文件夹的名字的意义在于，可以通过这种方式实现本地同时运行多个节点。它们会读取不同的配置文件，并创建互相独立的 db 数据。

### 启动

#### 通过一键脚本启动

启动 cosd 并正常出块的流程较为繁琐，如果只是需要启动并快速继续后面的内容，我们计划提供一键启动的脚本，可以完成多（四）节点启动并且正常出块。但是目前这部分并没有提供。
当然，如果是希望了解整个启动流程，我们建议使用非一键启动的方式。

#### 非一键启动

##### 单节点

初始化完成之后通过执行 `start` 命令启动 cosd。

```shell
cosd start
```

上述命令会使用 `~/.coschain/cosd` 下的配置文件，并且写入数据到这个文件夹下。

同样，也可以指定配置文件夹。

```shell
cosd start -n testnode
```

会使用 `~/.coschain/testnode` 下的配置文件，并且写入数据到这个文件夹下。

执行后，cosd 节点就应该可以正常启动了。

##### 多节点

即使单节点在本地成功运行，这是因为：

1. 按照 SABFT 协议要求，至少三个节点同时工作才能达成共识并且正常出块
2. 需要手动选择出块节点

###### 使用 multinodetester

虽然可以手动创建多个配置文件并启动，但是这个过程比较繁琐。对于此，我们提供了 multinodetester 用于批量创建和启动。

multinodetester 同样在 cmd 下。需要编译。

```shell
cd build
go build ../cmd/multinodetester
```

**批量初始化**

```shell
multinodetester init 4
```

可以初始化 4 个配置文件夹，可以在 `~/.coschain` 下找到，分别为 `testcosd_0` 到 `testcosd_3`， 对应 4 个节点。

**批量启动**

```shell
multinodetester start path/to/cosd 4
```

path/to/cosd 是 cosd 可执行文件的路线，如果 mltinodetester 和 cosd 都在 build 文件夹下，那么这个 path 就是 `./cosd`。

通过以上步骤，本地就会有 4 个节点运行。

###### 手动选择第一轮出块节点

本地即使有多个节点在运行，但是如果没有选择出块节点，那么默认只有 `testcosd_0` 这个特殊节点( initminer )出块，其他节点都处于同步模式。关于 initminer 节点以及block producer(bp) 节点和同步节点的区别，请参考 [基本概念](/zh-cn/tutorial/concept.md)。所以我们需要手动选择出块节点。

一般情况下，一个节点需要通过 wallet 去注册节点，节点投票之后才能成为为 bp 节点，这个过程同样繁琐。在本地环境下，简化了这个流程，在 wallet 下提供了同名 `multinodetester` 方法来批量注册和选举 bp 节点。

如果严格按照上文流程，那么现在本地应该有 `wallet-cli` 这个可执行文件。

```shell
wallet-cli
```

执行这个文件进入交互界面。输入 `multinodetester 4` 来批量注册选举 bp 节点。
也可以通过非交互模式完成，执行:

```shell
wallet-cli multinodetester 4
```

经过以上步骤，contentos 主链应该已经成功在本地运行起来，试着通过 [wallet 进行交互](/zh-cn/tutorial/wallet.md)。
不过在此之前，需要解释一些[基本概念](/zh-cn/tutorial/concept.md)。