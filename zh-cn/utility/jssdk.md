# js-sdk

## 介绍

contentos 的 js sdk，目前 `web-wallet` 和 `explorer` 都通过该 sdk 与 cosd 交互。

代码在 [cos-js-sdk](https://github.com/coschain/cos-sdk-grpc-js) 并且已经在 npm 上发布，可以通过 `npm install cos-grpc-js` 安装。

## 使用

sdk 基于 [grpc-web](https://github.com/improbable-eng/grpc-web)，`src/lib` 中 `prototype` 与 `grpc` 
包都来自于 `contentos-go` 中对应路径的 [protobuf](https://developers.google.com/protocol-buffers) 生成结果，不推荐修改。


具体可以通过阅读 [cos-web-toolkit](https://github.com/coschain/cos-web-toolkit/blob/master/src/encrypt/clientsign.js) 中相应代码查看调用方式。
