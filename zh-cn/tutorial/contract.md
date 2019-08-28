# 合约

## 本节将会学习到

1. 如何搭建合约编译环境
2. 一个简单的 hello world 合约
3. 编译合约与 ABI 文件
5. 部署并调用合约函数
6. 一个发币合约

## 概念

Contentos 合约使用 C++ 开发，链上直接执行的是一个 [WebAssembly](https://webassembly.org/) 文件。虽然其实各个语言都可以编译成 wasm 文件，但是目前比较成熟的还是 C++。
Contentos 合约执行的原理和 EOS 的方式十分类似，如果有过 EOS 合约开发的经验，上手会十分容易。

## 合约环境

合约环境和链运行的环境无关，准确地说，是提供了一个运行合约编译器的环境。正因为如此，Contentos 提供了一个 [网站](http://studio.contentos.io/) 用作在线编译合约。这样就不用配置繁琐的本地环境。

当然，如果是合约开发者，在本地安装合约环境依然是必要的。

### 第一步：获取 compier 源码

从 github 上获取 master 分支的代码

```shell
git clone git@github.com:coschain/wasm-compiler.git
```

### 第二步：编译

#### 在 ubuntu 系统安装合约环境

目前 github 上步骤过于山寨，需要讨论

#### 在 macOS 系统安装合约环境

缺失，要么步骤写全，要么打一个 homebrew 包

## hello world 合约

创建一个 contract 文件夹

```shell
mkdir Contracts
```

创建一个 hello 文件夹

```shell
cd Contracts
mkdir hello
cd hello
```

用你喜欢的编辑器创建一个新文件

```shell
touch hello.cpp
```

引用 `contract.hpp` ，它包括了一些写合约必须的类

```cpp
#include <cosiolib/contract.hpp>
```

引用 `print.hpp`，用来输出结果

```cpp
#include <cosiolib/print.hpp>
```

创建一个标准的 cpp 类，继承自父类 `cosio::contract`

```cpp
class hello : public cosio::contract {}
```

使用 using 来构造 contract

```cpp
class hello : public cosio::contract {
    public:
        using cosio::contract::contract;
}
```

定义一个 greet 函数

```cpp
class hello : public cosio::contract {
    public:
        using cosio::contract::contract;
        void greet() {
            cosio::print_f("Hello %\n", get_caller());
        }
}
```

`get_caller` 可以获取当前合约的调用者。

定义需要暴露到外界的接口

```cpp
COSIO_ABI(hello, (greet))
```

完整代码如下

```cpp
#include <cosiolib/contract.hpp>
#include <cosiolib/print.hpp>

class hello : public cosio::contract {
public:
    // all contract classes MUST have same constructor prototype
    using cosio::contract::contract;

    // contract method: greet
    void greet() {
      // say hello to contract caller
      cosio::print_f("Hello %\n", get_caller());
    }
};

COSIO_ABI(hello, (greet))
```

## 编译合约与 ABI 文件

### 编译合约

如果已经搭建了本地编译环境，并且创建了 hellp.cpp 那么可以通过:

```shell
cosiocc -o hello.wasm hello.cpp
cosiocc -g hello.abi hello.cpp
```

分别得到 .abi 文件和 .wasm 文件。

.wasm 文件是在链上实际被执行的合约代码，但是 WebAssembly 本身没有类型信息，这部分需要 abi 文件补足。

### ABI 文件

ABI 的全称是 The Application Binary Interface (ABI)。这个文件基于 json 描述，程序通过 ABI 文件判断方法的有无，以及确定链上返回数据的结构和类型。

如果是 hello 合约，编译出来的 abi 文件内容如下：

```json
{
    "actions": [
        {
            "name": "greet",
            "type": "greet"
        }
    ],
    "structs": [
        {
            "base": "",
            "fields": null,
            "name": "greet"
        }
    ],
    "tables": null,
    "types": null,
    "version": "contentos::abi-1.0"
}
```

上述 abi 文件描述了 hello 合约暴露出的接口 greet 。并且给定了 greet 函数的参数和类型。当然 hello 合约比较简单，更多的 abi 文件会在后文进行分析。

### 部署/调用合约

#### 部署

`deploy` 命令用以部署合约，命令格式如下

```shell
deploy [author] [contract_name] [local_wasm_path] [local_abi_path] [upgradeable]
```

所以假设是 initminer 用户部署 hello 合约，并且 .wasm 和 .abi 都在当前目录下。

```shell
./wallet-cli
> deploy initminer hello ./hello.wasm ./hello.abi true
```

**返回**

```shell
> deploy initminer hello ./hello.wasm ./hello.abi true
Result: invoice:<status:200 net_usage:44040 cpu_usage:1 op_results:<> >
```

#### 调用

`call` 命令用以调用合约，命令格式如下

```shell
call [caller] [owner] [contract_name] [method] [args]
```

因为合约名并不保证唯一，所以需要通过合约的拥有者以及合约名确定合约。

args 是一个列表，使用 json 格式，需要注意转义。

使用 initminer 调用 hello 合约的 greet 函数，形式如下

```shell
./wallet-cli
> call initminer initminer hello greet []
```

**返回**

```shell
> call initminer initminer hello greet []
Result: invoice:<status:200 op_results:<vm_console:"Hello initminer\n" > >
```

## 一个发币合约

这个示例合约可以在[这里](https://github.com/coschain/wasm-compiler/blob/master/contracts/token/token.cpp) 找到。

```cpp
#include <cosiolib/contract.hpp>
#include <cosiolib/print.hpp>
#include <cosiolib/system.hpp>

using namespace std;

//
// Token contract maintains 2 database tables.
// One is "stats" table, which contains a single row recording properties of the token.
// The other is "balances" table in which each row records a token account and its balance.
//

/**
 * @brief record type of "balances" table.
 */
struct balance {
    cosio::name tokenOwner;         ///< name of account who owns the token
    uint64_t amount;                ///< balance of the account

    // specify the sequence of fields for serialization.
    COSIO_SERIALIZE(balance, (tokenOwner)(amount))
};

/**
 * @brief record type of "stats" table.
 */
struct stat : public cosio::singleton_record {
    string name;                    ///< name of the token
    string symbol;                  ///< symbol name of the token
    uint64_t total_supply;          ///< total number of tokens issued
    uint32_t decimals;              ///< number of digits after decimal point

    // specify the sequence of fields for serialization.
    COSIO_SERIALIZE_DERIVED(stat, cosio::singleton_record, (name)(symbol)(total_supply)(decimals))
};

/**
 * @brief the token contract class
 */
struct cosToken : public cosio::contract {
    using cosio::contract::contract;

    /**
     * @brief contract method to create a new type of token.
     * 
     * @param name          name of the token, e.g. "Native token of Contentos".
     * @param symbol        symbol name of the token, e.g. "COS".
     * @param total_supply  total number of tokens to issue.
     * @param decimals      number of digits after decimal point, e.g. with decimals==3, 12345 COS tokens represent as "12.345 COS".
     */
    void create(string name,string symbol, uint64_t total_supply, uint32_t decimals) {
        // make sure that only the contract owner can create her token
        cosio::require_auth(get_name().account());

        // create the stats record in database with default record member values.
        stats.get_or_create();
        // update the stats record
        stats.update([&](stat& s){
                s.name = name;
                s.symbol = symbol;
                s.total_supply = total_supply;
                s.decimals = decimals;
                });

        // the contract owner owns all tokens
        // record it into the balance table
        auto owner = get_name();
        balances.insert([&](balance& b){
            auto account_name = owner.account();
            b.tokenOwner.set_string(account_name);
            b.amount = total_supply;
        });

        // [optional] query and print balance of contract owner.
        auto b = balances.get(owner.account());
        cosio::print_f("user % has % tokens. \n", b.tokenOwner.string(), b.amount);
    }

    /**
     * @brief contract method to transfer tokens.
     * 
     * @param from      the account who sends tokens.
     * @param to        the account who receives tokens.
     * @param amount    number of tokens to transfer.
     */
    void transfer(cosio::name from,cosio::name to, uint64_t amount) {
        // we need authority of sender account.
        cosio::require_auth(from);

        // check if sender has any tokens.
        cosio::cosio_assert(balances.has(from), std::string("no balance:") + from.string());
        // check if sender has enough tokens.
        cosio::cosio_assert(balances.get(from).amount >= amount, std::string("balance not enough:") + from.string());
        // check integer overflow
        cosio::cosio_assert(balances.get_or_default(to).amount + amount > balances.get_or_default(to).amount, std::string("over flow"));

        // total balance of both sender and receiver
        auto previousBalances = balances.get_or_default(from).amount + balances.get_or_default(to).amount;

        // decrease sender's balance
        balances.update(from,[&](balance& b){
                    b.amount -= amount;
                    });
        // increase receiver's balance
        if(!balances.has(to)) {
            balances.insert([&](balance& b){
                        b.tokenOwner = to;
                        b.amount = amount;
                    });
        } else {
            balances.update(to,[&](balance& b){
                       b.amount += amount;
                    });
        }

        // make sure that total balance of both accounts not changed.
        cosio::cosio_assert(balances.get(from).amount + balances.get(to).amount == previousBalances, std::string("balance not equal after transfer"));
    }

    //
    // define a class member named "balances" representing a database table which,
    // - has name of "balances", the same as variable name,
    // - has a record type of balance
    // - takes balance::tokenOwner as primary key
    //
    COSIO_DEFINE_TABLE( balances, balance, (tokenOwner) );

    //
    // define a class data member named "stats" representing a singleton table which,
    // - has name of "stats"
    // - has a record type of stat
    //
    COSIO_DEFINE_NAMED_SINGLETON( stats, "stats", stat );
};

// declare the class and methods of contract.
COSIO_ABI(cosToken, (create)(transfer))
```