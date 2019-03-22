# 本地环境搭建

## 获取源代码

Contentos 本身是完全开源，并且托管在 github 上，可以通过访问 [contentos-go](https://github.com/coschain/contentos-go) 了解项目状态。

可以通过任何 [git](https://git-scm.com/) 工具获得最新的代码。或者下载 [release 版本](https://github.com/coschain/contentos-go/releases)。

注意，contentos 主链目前处于测试状态，release 版本更迭频繁，可能会有一些破坏性的更改被添加。

## 依赖

contentos-go 依赖于 Golang 的 1.11.4 以上的版本，请查阅[Go 官方的文档](https://golang.org/) 以配置本地环境。目前 contentos-go 并没有对 windows 系统做适配或者优化，请使用 unix-like 系统。

## 构建

contentos-go 可构建出以下程序:

* cosd: contento-go 主链
* wallet-cli: 和链进行交互的客户端

如果已经 `git clone` 或者下载了源代码，请 `cd` 到该文件夹，通过

```bash
make build_cosd
```

以及 

```bash
make build_wallet
```

分别编译出两者。

### wallet-cli
用来和 `cosd` 进行交互，直接执行可进入交互模式，并通过 `8888` 端口和本地的 `cosd` 进程进行连接。目前提供了以下命令:

* account
* bp
* claim
* close
* create
* follow
* genKeyPair
* import
* info
* locked
* list
* load
* lock
* post
* reply
* transfer
* unlock
* transfer_vesting
* vote

通过执行 `--help` 来查询相应命令的说明和参数。
同时我们也提供了一个 web 的版本，请访问 [cos-web-toolkit](https://github.com/coschain/cos-web-toolkit) 查看。

### cosd

通过执行 `cosd init` 会在本地初始化环境，默认地址为 `$HOME/.coschain`。
初始化完成，执行 `cosd start`，contentos 链即在本地运行。默认会开启 `8080`, `8888`, `20338` 三个端口。请确定没有被占用。

### 注意
由于 contentos 本身协议限制需要至少三个以上节点，所以单节点会在 1024s 之后因为找不到节点而退出。



