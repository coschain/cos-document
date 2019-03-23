# web-wallet

## 介绍

`web-wallet` 是 contentos 自带的 `wallet-cli` 的 web 版本，用来和链进行交互。但是目前并没有完整提供所有 `wallet-cli` 的功能，支持操作如下:

* create account
* generate keystore file
* transfer
* post

目前测试链的 web-wallet 已经上线，可以访问 <https://testwallet.contentos.io/> 查看。

## 本地构建

web-wallet 全部代码托管在 github 上，访问 [cos-web-toolkit](https://github.com/coschain/cos-web-toolkit) 查看。

要求 `nodejs` 版本大于 `6.0.0`, `npm` 版本大于 `3.0.0`。

获取代码后，进入源代码根目录，`npm install` 安装依赖。

## 运行

### dev 环境
`web-wallet` 分为前端代码和后端代码两个部分，dev 环境需要开启两个服务。

`npm run dev && npm run debug_server` 注意 `package.json` 中，`debug_server` 的具体命令，配置了默认的 `CREATOR` 环境变量与值。
这个变量对应 cosd 中一个特殊用户 `accountcreator`, 这个用户和 initminer 不同，并不会在链首次启动的时候创建，而是需要你手动创建。

使用默认的私钥是危险的，如果是正式服务，请修改这个私钥。

### pm2 托管
也可以通过 pm2 运行。默认的 pm2 配置是 `pm2.config.js`，请注意 `error_file` 和 `out_file` 的地址可能需要修改。

执行 `npm run build` 之后通过 `CREATOR=yourPrivKey pm2 start pm2.config.js` 运行 web-wallet，`CREATOR` 请改为自己的值。

