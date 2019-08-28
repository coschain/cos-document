# 配置文件与一些概念

## 本节将会学习到

1. 配置文件中各属性的含义
2. 什么是 bp 节点
3. 什么是 initminer

## 配置文件

上一节运行过 `cosd init` 命令，这个命令会在 `~/.coschain` 下生成配置文件夹，该文件夹下存在主配置文件 `config.toml`。一般情况下不需要过多修改。

该配置文件分为若干节。最外层包括 `ChainId` 和 `Name`。

`ChainId` 相同的节点才能互相发现并连接。如果通过 `cosd init`，默认初始化的 ChainId 是 main，可以通过 `cosd init -n` 指定 Chaind。同样 multinodetester 默认初始化的 `ChainId` 也是 `main`，如果需要指定其他属性的链，需要使用 `-c` 指定，如: `multinodetester -c dev init 4`。

`Name` 用来指定节点的名字，节点名字对应应该读取本地哪一份配置文件。默认的名字是 cosd，该节点会读取 `~/.coschain/cosd` 这份配置文件。

### Consensus

`BootStrap` 如果被设置为 `true`，那么如果被选为了 bp 节点，在轮到自己出块的时候，会以 `LocalBpName` 的名义，用 `LocalBpPrivateKey` 签名产出的块。
`BootStrap` 如果被设置为 `false`，那么就只会使用同步模式。即使被选为了 bp 节点，这个节点在轮到自己出块的时候也不会出块。

### Database

这个配置是可选的，并且不被主链使用，只会在用户显式运行某些 [插件]() 的时候被使用。对应一个标准的 sql 数据库。目前只支持 mysql。

### GRPC

不建议修改。cosd 主要通过 grpc 向外界提供接口，包括 wallet 等都是通过 grpc 与 cosd 交互。cosd 会默认使用 8080 端口交互，并且在 8888 端口启动一个 http 的反向代理，用来把 http 的 grpc 请求响应转为标准的基于 socket 的 grpc 请求和响应。

### HealthCheck

healthcheck 端口

### P2P

不建议修改。

SeedList 用于指定种子节点，建议使用公开的种子节点。

## 一些基本概念

上一部分涉及了一些 coschain 的基本概念和术语，包括：

### bp 节点

coschain 的共识协议使用 [SaBFT]()。这个模型下，只有出块节点 (bp 节点) 才有出块的权利，非出块节点只能同步 bp 节点的数据。要成为 bp 节点，需要有足够的抵押，然后注册为 bp 节点，这时候这个节点才拥有成为 bp 节点的资格。拥有资格并且获得足够多的票数，一个节点才能真正成为 bp 节点，bp 节点每出一个块都能获得一定奖励。

### initminer

coschain 刚启动的时候并没有其他的 bp 节点，这时候由名字为 initminer 的节点出块。initminer 节点出块并不会产生奖励。一旦成功选出了其他 bp 节点，initminer 节点会退出 bp 列表，之后也不再加入。