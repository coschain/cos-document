# 智能合约

contentos 智能合约需要用`C++`语言进行编写，开发者可以通过在线编辑器 XXXXX 编写并编译 contentos 智能合约。

## 编写、编译及部署 contentos 智能合约的说明

### 1、编写合约

访问在线编辑器 XXXXX，在编写自己的合约之前，需要做以下修改：

首先，修改脚本`abi.ts`，将脚本如下内容中的`hello`改成自己的合约文件名，注意，该文件中有两处需要修改，分别是`hello.cpp`和`hello.abi`

```bash
  const data = await Service.compileAbi([project.getFile("src/hello.cpp"),project.getFile("src/header.hpp")], "cpp", "abi");
  const outAbi= project.newFile("out/hello.abi", "abi", true);
```

其次，修改脚本`build.ts`，将脚本如下内容中的`hello`改成自己的合约文件名，同样的，该文件中也有两处需要修改，分别是`hello.cpp`和`hello.wasm`
```bash
  const data = await Service.compileFile([project.getFile("src/hello.cpp"),project.getFile("src/header.hpp")], "cpp", "wasm", "-g -O3");
  const outWasm = project.newFile("out/hello.wasm", "wasm", true);
```

最后，将最左侧`src`路径下的`hello.cpp`文件名改成自己的合约文件名（点击鼠标右键进行编辑）

### 2、编译合约

点击最上方的`Abi`按钮编译合约 abi 文件，点击`Build`按钮编译合约 wasm 文件，编译过程中如遇到错误，会在下方的窗口中显示，点击`Download`按钮可以将生成好的 abi 及 wasm 文件下载到本地

### 3、合约部署及调用

命令行工具 `wallet-cli` 的 `deploy` 和 `call` 命令可以进行合约的部署及调用，`wallet-cli` 的使用方法请参考文档 [钱包使用说明](https://github.com/coschain/contentos-go/blob/master/README.md)
