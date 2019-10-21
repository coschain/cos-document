#  contract native functions

## Overview 

The contracts of coschain are written by cpp language, and pure cpp of course cannot interact with coschain that why we provide native functions which can interact with coschain. Though these, the contract writer can query current chain state or launch a transfer transaction in contract. These functions can be found in [system.hpp](https://github.com/coschain/wasm-compiler/blob/master/contracts/cosiolib/system.hpp) and I list them below with some brief explains.

## Native Functions

### current_block_number 

It returns current block number

**Declaration**

```cpp
uint64_t current_block_number();
```

**Return**

current block number with type uint64_t

### current_timestamp

It returns current UTC timestamp in coschain.

**Declaration**

```cpp
uint64_t current_timestamp();
```

**Return**

current UTC timstamp

### current_block_producer

It returns the account name of current block producer.

**Declaration**

```cpp
std::string current_block_producer();
```

**Return**

Current block producer's account name

### block_producers

Current block producers

**Declaration**

```cpp
std::vector<std::string> block_producers();
```

**Return**

A vector contains all current top21 block producers.

### sha256

It returns sha256 value of argument.

**Declaration**

```cpp
checksum256 sha256(const bytes& data);
```

**Return**

sha256 value of argument.

### get_contract_name

It returns current contract name

**Declaration**

```cpp
name get_contract_name();
```

**Return**

Return name of current contract. The defination of name can be found in [types.hpp](https://github.com/coschain/wasm-compiler/blob/master/contracts/cosiolib/types.hpp)

### is_contract_called_by_user

Check whether its called by user or another contract.

**Declaration**

```cpp
bool is_contract_called_by_user();
```

**Return**

True refers its called by a user otherwise by a contract

### get_contract_caller

It returns the caller of current calling in contract

**Declaration**

```cpp
name get_contract_caller();
```

**Return**

If it was called by a user, returns his/her account name otherwise returns the contract name.

### get_contract_method

To know the method name in this calling.

**Declaration**

```cpp
std::string get_contract_method();
```

**Return**

The method name in this calling

### get_contract_args

To know the argument in this calling

**Declaration**

```cpp
bytes get_contract_args();
```

**Return**

Return a datastream which refers the arguments encoded by datastream, more info in [datastream.hpp](https://github.com/coschain/wasm-compiler/blob/master/contracts/cosiolib/datastream.hpp).

### get_contract_balance

It returns a contract's balance

**Declaration**

```cpp
coin_amount get_contract_balance(const name& contract);
```

**Parameter**

The contract's name

**Return**

Returns balance of a contract with type coin_amount. About coin_amount, it can be found in [types.hpp](https://github.com/coschain/wasm-compiler/blob/master/contracts/cosiolib/types.hpp) which is a alias of uint64_t.

### get_user_balance

It returns a user's balance

**Declaration**

```cpp
coin_amount get_user_balance(const name& user);
```

**Parameter**

The parameter user should be a user's name rather than a contract's, otherwise it abort execution.

**Return**

Return balance of a user.

### user_exist

Check whether a user exist.

**Declaration**

```cpp
bool user_exist(const name& user);
```

**Parameter**

The parameter user should be a user's name rather than a contract's, otherwise it abort execution.

**返回值**

1 for user existing, or 0 for not.

### get_balance

It returns a balance despite a user or contract.

**Declaration**

```cpp
coin_amount get_balance(const name& who)
```

**Parameter**

A name belongs to a user or a contract.

**Return**

The balance of a user or a contract with coin_amount type.

### get_contract_sender_value

It reads sending amount cos by caller.

**Declaration**

```cpp
coin_amount get_contract_sender_value()
```

**Return**

A caller can send some COS to contract when calling contract's method. And contract read it by the method.

### require_auth

Assert whether a user has authority to specific operation.

**Description**

* Different operation need different authority, like a transfer operations, it need be signed by the sender of transfer and so on. The method can be used in this situation. 
* If the caller is a contract, it only verify whether the caller is some specific names.

**Declaration**

```cpp
void require_auth(const name& who)
```

**Parameter**

Who is a name for a user or a contract.

### transfer_to_user

Transfer to a user from the contract.

```cpp
void transfer_to_user(const name& to, coin_amount amount, const std::string& memo);
```

**Declaration**

```cpp
void transfer_to_user(const name& to, coin_amount amount, const std::string& memo);
```

**Parameter**

* to should be a user's name otherwise it abort execution.
* amount is the type of coin_amount.
* memo can be a empty string.

### transfer_to_contract

Transfer to a contract from a contract

**Declaration**

```cpp
void transfer_to_contract(const name& to, coin_amount amount, const std::string& memo);
```

**Parameter**

* to should be a user's name otherwise it abort execution.
* amount is the type of coin_amount.
* memo can be a empty string.

### transfer_to

It automatically chooses whether calls trasfer_to_user or transfer_to_contract, by *to* name.

**Declaration**

```cpp
void transfer_to(const name& to, coin_amount amount, const std::string& memo);
```

**Parameter**

* to should be a user's name otherwise it abort execution.
* amount is the type of coin_amount.
* memo can be a empty string.串

### execute_contract

Calling another contract's method in contract

**Declaration Style 1**

```cpp
void execute_contract(const name& contract, const std::string& method, const bytes& params, coin_amount coins);
```

**Declaration Style 2**

```cpp
template<typename...Args>
static void execute_contract( const name& contract, const std::string& method, coin_amount coins, Args...args );
```

**Parameter**

In style 1

* contract should be a contract's name
* params is a bytes stream encoded by datastream refer to arguments. Read [datastream.hpp](https://github.com/coschain/wasm-compiler/blob/master/contracts/cosiolib/datastream.hpp) to know more.

In style 2:

* args should be the arguments.


### reputation_admin

Acquire current reputation admin

**Description**

In coschain, there is a special user called reputation admin who can set a user's reputation. If reputation were set to 0, the user cannot get any reward in coschain. The method can acquire account name of the reputation admin.

**Declaration**

```cpp
std::string reputation_admin();
```

**Return**

Account name of the reputation admin
