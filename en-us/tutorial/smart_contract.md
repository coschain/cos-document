# Smart contract



## 1. write a smart contract in cpp

The following steps show how to write a demo token contract from scratch. The contract `cosToken` provides two actions: `create and transfer`.  The owner who deploys this contract needs to call `create` exactly once first to create his/her own token,  then `transfer` can be called by all other users to transfer the token, provided users have sufficient balance.

NOTE: this is just a **DEMO** to illustrate how to build a smart contract on Contentos, it's **NOT** a general token issuing smart contract.

### 1.1  import library and namespace

```cpp
#include <cosiolib/contract.hpp> // This is essential for any smart contract
#include <cosiolib/print.hpp> // Used to emit log
#include <cosiolib/system.hpp> // It's required to have access to system functions. In this case require_auth and cosio_assert is called

using namespace std;
```

Other header files might be imported if needed.



###1.2  define table struct 

Table structs are defined if a smart contract need to stored structured data on blockchain. In this case 2 tables are needed. One is `stats` table, which contains a single row recording properties of the token. The other is `balances` table in which each row records a token account and its balance.

```cpp
/**
 * @brief record type of "balances" table.
 */
struct balance {
    cosio::name tokenOwner;         ///< name of account who owns the token
    uint64_t amount;                ///< balance of the account

    // specify the sequence of fields for serialization.
    COSIO_SERIALIZE(balance, (tokenOwner)(amount))
};
```

`COSIO_SERIALIZE` specifies the fields and in what order they are serialized. If a field is omitted, it will not be stored on chain.



```cpp
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
```

Since `stat` is derived from `cosio::singleton_record`, here  `COSIO_SERIALIZE_DERIVED` is called instead. It means that the base class `cosio::singleton_record` is serialized first before the fields.  



### 1.3  define token class

```cpp
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
```

`using cosio::contract::contract;` must be added in the contract class.  It defines the contract's constructor and it's underlying owner and caller. 

*  `get_name().account()` returns the owner of the contract

* `require_auth(account_name)` asserts that the transaction containing the action needs to be signed by `account_name`
* table provides following API:  `name(), has(), get(), insert(), update(), remove(), get_or_default(), get_or_create()`, there meanings are pretty straightforward.



###1.4 declare the class and methods of contract.

```cpp
COSIO_ABI(cosToken, (create)(transfer)) // specifies that the contract exposes two actions: create and transfer
```



## 2. Build your smart contract

An online IDE is provided [here](http://studio.contentos.io/).  Just start a token project, above source code will be generated automatically for you. Press the `Build` button and the IDE will do the work for you. When it's done,  there will be a `wasm` file and a `abi` file generated.

![build](https://github.com/coschain/cos-document/tree/master/en-us/tutorial/contract_build.jpg)

## 3. deploy your contract on contentos blockchain

To deploy your smart contract, just press `Deploy` button, then fill in the first 3 blanks in the following picture and hit `Confirm` button. Your contract will be deployed on our testnet.

![deploy](https://github.com/coschain/cos-document/tree/master/en-us/tutorial/contract_deploy.jpg)



## 4. Interact with smart contract

### 4.1 wallet

Users can invoke contract action via Contentos wallet

```shell
call [caller] [owner] [contract_name] [method] [args]
```

Say after the deployment of the contract, `initminer` wants to create his token "COC"

```shell
./wallet-cli
> call initminer initminer cosToken create "[\"COS coin\", \"COC\", 10000000, 3]"
```

To transfer 1 COC from` initminer` to `alice`, 

```shell
> call initminer initminer cosToken transfer "[\"initminer\", \"alice\", 1]"
```



### 4.2 website

Users can also visit [here](https://testwallet.contentos.io/#/create)

#### 4.2.1 import key

![import_key](https://github.com/coschain/cos-document/tree/master/en-us/tutorial/import_key_web_wallet.jpg)

#### 4.2.2 invoke contract action

![create](https://github.com/coschain/cos-document/tree/master/en-us/tutorial/action_web_wallet.jpg)
