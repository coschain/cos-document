# 使用 wallet 和链交互

## 本节将会学习到

1. 账户相关
    1. 账户列表与账户详情
    2. 从私钥导入账户
    3. 通过 wallet 创建账户
    4. 生成助记词，通过助记词创建账户
    5. 导入本地账户/锁定/解锁
2. 流通相关
    1. 转账
    2. 抵押
    3. 抵押获取耐力值
3. 内容相关
    1. 发表内容
    2. 回复内容
    3. 投票

### 注意

以下所有的命令与返回，都在 wallet-cli 的交互模式下完成。请确保本地公链已经启动并正常出块。
所有的命令都可以后跟 `--help` 了解用法。

## 账户相关

### 概念

contentos 账户分为本地账户和链上账户。

### 本地账户

本地账户是一份 keystore 文件，被放置在 `~/.coschain` 文件夹下。一份 keystore 存储了一对 contentos 的公私钥。keystore 文件可以被 wallet 创建或者导入。
一份本地的 keystore 类似于:

```json
{  
   "Name":"testuser",
   "PubKey":"COS745h9zeER6qea8q9GVnRVvia7dHXnbPSAKvF84SbwSdzBG5pa1",
   "Cipher":"AES-256",
   "CipherText":"Bo3DR/lPtrfau4CsgHBFR4jPE78fPUnY5xY9pO6OKhuMe5fTZ2jxPd8MJDzcqiFSf7o=",
   "Iv":"Bj8mdb0z8JAFrAdvNSl+Sw==",
   "Mac":"sgZbYnWPF26v4cWbhIA6BhMKOxvEVPm/NwEpcViHkYE=",
   "Version":1
}
```

Name 对应链上账户的名称，公钥公开存放，私钥加密存放。

### 链上账户

如果一个账户在链上被创建成功，那么可以通过命令（见下文）查询到具体信息。链上账户包含丰富的信息，包括账户余额，抵押数量等。
链上账户通过 wallet 创建。

### 账户列表与详情

**查询本地账户**

```shell
./wallet-cli
> list
```

**返回**

```shell
> list
account: initminer15 | status:   locked
account:  initminer2 | status:   locked
account:  initminer5 | status:   locked
account: initminer11 | status:   locked
account: initminer16 | status:   locked
```

**查看已解锁账户公私钥**

```shell
./wallet-cli
> info initminer10
```

**返回**

```shell
> info initminer10
account: initminer10
pub_key: COS8fV44EgGUWjEKddPbfrXAaqYxYes4M99KLHYBwrXEj3hWP4XPr
priv_key: 4UKbH7TDr6FgFXNjzQdX3ucAXndzY4h7YBNNgnqjtUeLJkFXVg
status: unlocked
```

**查看账户在链上的信息**

```shell
./wallet-cli
> account get initminer
```

**返回**

```shell
> account get initminer
GetAccountByName detail: {
	"info": {
		"account_name": "initminer",
		"coin": "6499969988.999988 COS",
		"vest": "0.000000 VEST",
		"public_key": "COS5JVLLcTPhq4Unr194JzWPDNSYGoMcam8yxnsjgRVo3Nb7ioyFW",
		"created_time": "1970/1/1 00:00:00",
		"trx_count": 19,
		"vote_power": 1000,
		"stamina_free_remain": 95304,
		"stamina_stake_remain": 864000000000,
		"stamina_max": 864000100000,
		"stake_vest_for_me": "11.000000 VEST",
		"withdraw_remains": "0.000000 VEST",
		"withdraw_each_time": "0.000000 VEST",
		"next_withdraw_time": "1970/1/1 00:00:00",
		"bp_vote_count": 1,
        ...
```

### 通过私钥导入账户

如果已知一个链上账户的账户名与私钥，但是本地没有生成 keystore 文件的情况下，需要使用 `import` 命令导入私钥并生成 keystore 文件。常见于更换机器的情况。

```shell
./wallet-cli
> import initminer 4DjYx2KAGh1NP3dai7MZTLUBMMhMBPmwouKE8jhVSESywccpVZ
```

在本地环境下，initminer 的私钥是已知的，可以通过上述命令导入 initminer 账户。

### 通过 wallet 创建账户

和 EOS 类似，用户并不能自己创建账户，只能通过已经有账户的用户创建。

在本地环境下，initminer 账户有足够的余额来创建账户，导入即可。

```shell
./wallet-cli
> create initminer testuser1
```

上述命令会要求输入一个密码，以用来给私钥加密。请不要使用容易被攻击的密码，并保管好自己的 keystore。

如果尝试过创建 `alice` 账户，那么会返回失败，contentos 对账户长度和格式都有一定的限制。具体而言要求长度在 6 - 16 字符之间，支持小写字母和数字。
运行 `create` 命令会创建一个链上账户，并在本地创建一个 keystore 文件。
账户的公私钥是随机生成的，可以通过 `info` 命令查询公私钥详情。

### 生成助记词，通过助记词创建账户

contentos 实现了标准的 [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) 协议，这个协议允许你通过一组助记词生成公私钥对。

首先需要生成助记词:

```shell
./wallet-cli
> genKeyPair
Mnemonic: donate shaft leisure fashion basic oyster knock saddle broken prevent junior entire cake slice payment always direct system forum require smile bus follow expire
Public  Key:  COS7hGNoSNyAwUZ3kgkjFa4RCmsjcuZNxS2q2mfhMMML6CYZ5Dtqs
Private Key:  3PMfB7RY5irkcKTzRtWje3G4mhKRiGHYrriKDxyshxNdN4eAkN
```

这组助记词 "donate shaft leisure fashion basic oyster knock saddle broken prevent junior entire cake slice payment always direct system forum require smile bus follow expire" 会始终对应这个公私钥对。

然后通过命令

```shell
./wallet-cli
> create_from_mnemonic initminer testuser2
```

创建账户。

### 导入本地账户/锁定/解锁

本地账户通过 list 命令查询状态的时候，主要有 `lock` 状态和 `unlock` 状态，`lock` 状态下读取不了私钥，所以所有的请求都不能发起。
想要使用账户发送请求，需要 unlock。

```shell
./wallet-cli
> unlock initminer
```

输入设定的密码可以解锁。
对已解锁的账户，也可以通过 `lock` 命令让账户进入锁定状态。

wallet 启动的时候，会默认导入 `~/.coschain` 下的所有 keystore 文件。如果有其他路径的 keystore 文件，可以通过 `load` 命令导入。导入的账户，都是锁定状态。需要手动解锁。

对一定时间内没有操作的账户，wallet 会自动让其进入锁定状态，避免被外人窃取已解锁账户的公私钥。

## 流通相关

### 概念

contentos 中流通货币或者说本币为 COS，COS 可以通过抵押操作变成 VEST，VEST 不能流通。并且 COS 也可以通过抵押为 STAKING 换取耐力值。

### 转账

```shell
./wallet-cli
> transfer initminer testuser0 1.000000
```

COS 精确到六位小数。sender 和 receiver 不能是同一个人，并且 sender 应该是已解锁状态。

**返回**

```shell
> transfer initminer runetressa 1.000000
Result: invoice:<status:200 net_usage:1250 cpu_usage:1 op_results:<> >
```

### 抵押

```shell
./wallet-cli
> transfer_vest initminer testuser0 1.000000 'memo'
```

这个命令把 COS 转成 VEST，sender 和 receiver 可以是同一个人，COS 和 VEST 的汇率恒定为 1, Memo请根据实际情况替换。

**返回**

```shell
> transfer_vest initminer initminer 1.000000
Result: invoice:<status:200 net_usage:1240 cpu_usage:1 op_results:<> >
```

### 抵押获取耐力值

处理交易会消耗 bp 节点的资源，比如带宽资源和 cpu 资源，为了防止公地危机，用户交易是有消耗的。ETH 的做法是消耗 gas --- 少量的 ETH 代币。contentos 并没有采取这样的方案，而是有一套耐力机制。
每天的耐力值是有限的，并会因为执行交易而被消耗，直到耐力值不足以支撑交易的消耗。
每个账户都会有免费的耐力额度，足够日常所需。如果额度不够，可以通过抵押 COS 获取更多的耐力值额度。
耐力值会自动回复，一天会回复满所有耐力。

```shell
./wallet-cli
> stake initminer testuser0 1.000000
```

和 transfer_vest 相似，stake 命令允许 sender 和 receiver 是同一个账号。

**返回**

```
> stake initminer initminer 1.000000
Result: invoice:<status:200 net_usage:1250 cpu_usage:1 op_results:<> >
```

## 内容相关

### 概念

作为一条内容公链，用户可以通过发表内容来获取收益，相应的，wallet 也提供了命令去完成这些操作。
仅仅是发布内容，用户并不会有机会获取奖励，用户还需要获得投票。投票的数量和权重决定了奖励的多少，而更好的内容更可能获得更多的投票。

### 发表内容

```shell
./wallet-cli
> post initminer "article" "hello world" "hello world for everyone"
```

**返回**

```shell
> post initminer "article" "hello world" "hello world for everyone"
Result: invoice:<status:200 net_usage:1630 cpu_usage:1 op_results:<> >
```

### 发布回复

和 post 相似，回复也是一种内容，因此回复也可以获得奖励。同时，回复能获取的奖励也更少。

```shell
./wallet-cli
> reply "intminer" "content" 1571218688384377456
Result: invoice:<status:200 net_usage:1630 cpu_usage:1 op_results:<> >
```

其中 1571218688384377456 指代一篇链上 post 的 id，也可能是 reply 的 id，不过 reply 其实也只是一种特殊的 post。

### 投票

```shell
./wallet-cli
> vote "initminer" 1571218688384377456
Result: invoice:<status:200 net_usage:1630 cpu_usage:1 op_results:<> >
```

同样，1571218688384377456 指代一篇链上 post 或者 reply 的 id。
