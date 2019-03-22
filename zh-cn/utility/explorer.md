# explorer

## 介绍

explorer 是 contentos 的区块链浏览器，用来展示当前链的运行状态，目前测试链的 explorer 已经上线，可以访问 <http://explorer.contentos.io> 查看。

## 本地 contentos 监控

explorer 可以用来查看本地链的状态，有以下方法：

### customize rpc

<http://explorer.contentos.io> 提供了 `change rpc` 的功能，本地运行 cosd 之后，可以通过修改 rpc 到 `http://0.0.0.0:8080` 来查看本地链的状况

### 本地构建

explorer 全部代码托管在 github 上，访问 [explorer](https://github.com/coschain/block-explorer) 查看。

要求 `nodejs` 版本大于 `4.0.0`, `npm` 版本大于 `3.0.0`。

获取代码后，进入源代码根目录，`npm install` 安装依赖，`npm run dev` 运行测试环境，explorer 会默认从 <http://localhost:8080> 读取数据，如果本地 cosd 正常运行，那么就可以通过 <http://localhost:9091> 查看运行状态。
