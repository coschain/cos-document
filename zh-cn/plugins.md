# 插件

## 概述

插件是一些可选的服务，用来提供额外的数据或者功能（见下文）。它们和 cosd 主程序一同运行，需要在 cosd 启动时指定开启，默认不启动。部分服务依赖其他服务的运行。

大多数插件依赖标准数据库，尤其是 [mysql](https://www.mysql.com/)，推荐版本 5.8 以上。需要修改 ~/.coschain 下相应的数据库配置使其指向自己的配置，见 config.toml 的 `[Database]` 节。

## 目前提供的插件

### trxsqlservice

提供了把所有 contentos transactions 写入数据库的功能。

#### 启动

```bash
./cosd --plugin=trxsqlservice
```

#### 说明

需要在运行之前通过 `cosd db init` 初始化数据库表。

### block_log

提供了订阅状态变化的功能。与 trxsqlservice 不同，block_log 可以监测动态的数据变化，比如一次合约调用导致的账户余额的变化；以及可以监测系统内变量的变化，比如某个块产生的奖励。

block_log 目前只提供了以下信息的订阅:

1. 创建账户
2. 经济系统
3. 账户余额变化，包括账户的 balance 和 vest balance。并且包括合约账户的余额变化。
4. 抵押 COS 为 stake 的记录
5. 给 block producer 的投票
6. 转账记录

不同的信息会放入不同的表中供查阅。

#### 启动

```bash
./cosd --plugin=block_log_svc --plugin=block_log_proc_svc
```

### dailystatservice

提供了 dapp 各项信息的查阅，只有当这个服务启动时，/grpcpb.ApiService/GetDailyStats 接口才能提供服务。

#### 启动

```bash
./cosd --plugin=dailystatservice
```

#### 说明

dailystatservice 服务依赖于 trxsqlservice 和 block_log 服务的正常运行，所以正常工作的启动命令如下:

```bash
./cosd --plugin=block_log_svc --plugin=block_log_proc_svc --plugin=trxsqlservice --plugin=dailystatservice
```