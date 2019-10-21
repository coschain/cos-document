# 合约的 native 函数

## 概述

合约的执行最终的目的是和链进行交互，coschain 提供了一些 cpp 的方法，通过调用这些方法，合约代码能查询链的状态，能与链发生交互。这些方法，被称为 native 方法。所有的方法可以在 [system.h](https://github.com/coschain/wasm-compiler/blob/master/contracts/cosiolib/system.h) 以及 [system.hpp](https://github.com/coschain/wasm-compiler/blob/master/contracts/cosiolib/system.hpp) 中查询到。主要分为两个版本，C 版本和 CPP 版本，CPP 版本的 native 函数实际上是 C 版本的简单封装。

## Native Functions

### current_block_number 

和 C 版本的同名方法一样，返回当前的 block number

**声明**

```cpp
uint64_t current_block_number();
```

**返回值**

当前的 block number

### current_timestamp

和 C 版本同名方法一样，返回当前链上的 UTC 的 timestamp

**声明**

```cpp
uint64_t current_timestamp();
```

**返回值**

当前链的 UTC 的 timestamp

### current_block_producer

和 C 版本同名方法一样，这个方法会返回当前的 block producer 的 account name

**声明**

```cpp
std::string current_block_producer();
```

**返回值**

当前出块 bp 的 account name

### block_producers

返回当前所有的出块节点

**声明**

```cpp
std::vector<std::string> block_producers();
```

**返回值**

当前所有出块节点的 account name 列表

### sha256

返回传入值的 sha256

**声明**

```cpp
checksum256 sha256(const bytes& data);
```

**返回值**

传入值的 sha256

### get_contract_name

返回当前 contract 的名字

**声明**

```cpp
name get_contract_name();
```

**返回值**

当前 contract 的 name。name 类型的定义可以在 [types.hpp](https://github.com/coschain/wasm-compiler/blob/master/contracts/cosiolib/types.hpp) 中查看。

### is_contract_called_by_user

判断这个合约调用是被用户调用或者合约调用

**声明**

```cpp
bool is_contract_called_by_user();
```

**返回值**

true 代表被用户调用， false 代表被合约调用

### get_contract_caller

获取本次合约被谁调用，返回 account name

**声明**

```cpp
name get_contract_caller();
```

**返回值**

如果是用户调用，那么返回这个用户的 name，否则返回合约的 name。

### get_contract_method

获取本次调用的方法名

**声明**

```cpp
std::string get_contract_method();
```

**返回值**

返回本次调用的方法名

### get_contract_args

获取本次合约调用的参数

**声明**

```cpp
bytes get_contract_args();
```

**返回值**

返回本次合约调用传递的参数，这个实际上是一个被 datastream 编码的字节流，datastream 部分见 [datastream.hpp](https://github.com/coschain/wasm-compiler/blob/master/contracts/cosiolib/datastream.hpp).

### get_contract_balance

获取一个合约的 balance

**声明**

```cpp
coin_amount get_contract_balance(const name& contract);
```

**参数**

contract 的 name

**返回值**

返回这个合约的 balance，coin_amount 类型也可以在 [types.hpp](https://github.com/coschain/wasm-compiler/blob/master/contracts/cosiolib/types.hpp) 中查看。本质上就是个 uint64_t。

### get_user_balance

获取一个用户的 balance

**声明**

```cpp
coin_amount get_user_balance(const name& user);
```

**参数**

user 的 name，如果传入的是一个合约的 name，那么会断言出错，合约执行中断

**返回值**

返回这个用户的 balance，coin_amount 类型

### user_exist

判断一个用户是否存在

**声明**

```cpp
bool user_exist(const name& user);
```

**参数**

user 的 name，如果传入的是一个合约的 name，那么会断言出错，合约执行中断

**返回值**

返回 1 说明用户存在，0 说明不存在

### get_balance

返回账户的余额，无论是用户还是合约

**声明**

```cpp
coin_amount get_balance(const name& who)
```

**参数**

一个用户或者合约的 name

**返回值**

用户或者合约的 balance，coin_amount 类型

### get_contract_sender_value

读取调用者传来的金额

**声明**

```cpp
coin_amount get_contract_sender_value()
```

**返回值**

合约方法调用者可以向合约付款，这个函数的返回值就是这笔款项的大小

### require_auth

断言这个合约有特定账户的权限。

**说明**

* 各种 trx 都是需要签名的，比如如果是一个 transfer 交易，那么需要验证这个交易是否有发送者的签名。这样的需求，可以通过这个函数验证。这是对用户调用的情况。
* 如果是合约调用，那么只验证调用方是否是特定合约。

**声明**

```cpp
void require_auth(const name& who)
```

**参数**

who 是一个合约或者用户的名字

### transfer_to_user

合约给用户转账，转账金额从合约账户支出

```cpp
void transfer_to_user(const name& to, coin_amount amount, const std::string& memo);
```

**声明**

```cpp
void transfer_to_user(const name& to, coin_amount amount, const std::string& memo);
```

**参数**

* to 是一个用户的名字，不能是合约名，否则会断言出错
* amount 是 coin_amount 类型，代表 unit，1 VEST = 100000 unit
* memo 可以为空字符串

### transfer_to_contract

合约给合约转账，转账金额从合约账户支出

**声明**

```cpp
void transfer_to_contract(const name& to, coin_amount amount, const std::string& memo);
```

**参数**

* to 是一个用户的名字，不能是合约名，否则会断言出错
* amount 是 coin_amount 类型，代表 unit，1 VEST = 100000 unit
* memo 可以为空字符串

### transfer_to

合约转账，实际上是 transfer_to_user 和 transfer_to_contract 的封装，程序会根据传入的 to 自动选择调用哪一个函数

**声明**

```cpp
void transfer_to(const name& to, coin_amount amount, const std::string& memo);
```

**参数**

* to 是一个用户的名字，不能是合约名，否则会断言出错
* amount 是 coin_amount 类型，代表 unit，1 VEST = 100000 unit
* memo 可以为空字符串

### execute_contract

可以在合约中调用另一个合约的参数

**声明1**

```cpp
void execute_contract(const name& contract, const std::string& method, const bytes& params, coin_amount coins);
```

**声明2**

```cpp
template<typename...Args>
static void execute_contract( const name& contract, const std::string& method, coin_amount coins, Args...args );
```

**参数**

声明 1 中:

* contract 必须是一个合约的名字
* params 是一个序列化之后的字节数组，代表调用参数，序列化方法可以见 [datastream.hpp](https://github.com/coschain/wasm-compiler/blob/master/contracts/cosiolib/datastream.hpp)。不过一般而言，使用封装之后的声明 2 。

声明 2 中:

* args 即为调用参数


### reputation_admin

获取到当前的 reputation admin

**说明**

coschain 中有一个特殊的管理员被称为 reputation admin，它可以设置用户的信誉值。如果用户信誉值被设定为 0，那么用户将不会获取任何发帖或者点赞的收益。

**声明**

```cpp
std::string reputation_admin();
```

**返回值**

当前的 reputation admin 的名字
