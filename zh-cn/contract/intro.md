# 智能合约

contentos 主链已经支持智能合约的编写，但是相关工具还处于早期阶段。目前测试网并不支持智能合约的部署与执行，可以通过本地 `wallet-cli` 的 `deploy` 和 `call` 命令验证。

contentos 支持 [wasm](http://webassembly.org.cn/) 格式的智能合约，通过修改过的 [WebAssemblyStudio](https://github.com/coschain/WebAssemblyStudio) 把 cpp 代码编译为 wasm 文件。

相关工具和支持将会在之后的更新中加入。
