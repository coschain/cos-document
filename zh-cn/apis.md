# 链 Api

## 概述

Contentos 链使用 [grpc](https://grpc.io/) 提供公开的 APIs。
对于如果是直接的 grpc 字节流请求，contentos 链提供的服务端口为 8888，如果是通过 web 发起 http 请求，那么需要发送到 8080 这个反向代理端口，contentos 会把发送至这个端口的请求解析为 grpc 字节流，并把响应包装成 http 响应返回。这部分内容请参考: [grpc-web](https://github.com/improbable-eng/grpc-web)。
contentos 官方提供的[线上钱包](https://github.com/coschain/cos-web-toolkit)，使用了 grpc-web 技术。
grpc-web 并不是一个被广泛使用的方案，以下内容以原始 grpc 为准。

## 环境配置

和 http 接口不同，grpc 需要已知请求和响应的结构。contentos 所有的 grpc 接口定义和返回都是[公开](https://github.com/coschain/contentos-go/blob/master/rpc/pb/grpc.proto)的。

grpc 文件依赖于其他几个文件提供类型信息：[type.proto](https://github.com/coschain/contentos-go/blob/master/prototype/type.proto)，[multi_id.proto](https://github.com/coschain/contentos-go/blob/master/prototype/multi_id.proto)，[operation.proto](https://github.com/coschain/contentos-go/blob/master/prototype/operation.proto)，[transaction.proto](https://github.com/coschain/contentos-go/blob/master/prototype/transaction.proto)。

获取 grpc.proto 文件，type.proto 文件，multi_id.proto 文件，operation.proto 文件，transaction.proto 文件。通过 [grpc](https://grpc.io/) 提供的命令构建本地客户端，就可以请求链上数据。

## 约定

以下用 "/grpcpb.ApiService/" + grpc 函数名表示对应的 api 接口。
比如 "/grpcpb.ApiService/GetChainState" 表示 `rpc GetChainState (NonParamsRequest) returns (GetChainStateResponse)` 这个 grpc 接口。
使用 go 的语法。
部分结构输出需要辅助函数处理，请参考 [prototype 文件夹](https://github.com/coschain/contentos-go/tree/master/prototype)下函数定义。

## Apis

### /grpcpb.ApiService/GetChainState

这个接口能查询到当前链的状态

#### 请求

NonParams

#### 返回

**GetChainStateResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| last_irreversible_block_number | uint64 |  |  |
| last_irreversible_block_time | uint64 |  |  |
| dgpo | prototype.dynamic_properties |  |  |

#### 代码示例

```go
func main() {
	conn, _ := grpc.Dial("localhost:8888",  grpc.WithInsecure())
	defer conn.Close()
	client := grpcpb.NewApiServiceClient(conn)
	req := &grpcpb.NonParamsRequest{}
	resp, _ := client.GetChainState(context.Background(), req)
	fmt.Println(resp)
}
```

### /grpcpb.ApiService/GetBlockList


这个接口能获取到两个 block height 之间的块信息。

#### 请求

**GetBlockListRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| start | uint64 |  |  |
| end | uint64 |  |  |
| limit | uint32 |  |  |

#### 返回

**GetBlockListResponse**


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| blocks | BlockInfo | repeated |  |

#### 代码示例

```go
func main() {
	conn, _ := grpc.Dial("localhost:8888",  grpc.WithInsecure())
	defer conn.Close()
	client := grpcpb.NewApiServiceClient(conn)
	req := &grpcpb.GetBlockListRequest{Start: 1, End: 100, Limit: 100}
	resp, _ := client.GetBlockList(context.Background(), req)
	fmt.Println(resp)
}
```

#### 说明

* start 是开始查询块的高度
* end 是结束查询块的高度
* end 设置为 0 表明当前块高度
* limit 最大只能为 100
* start 和 end 差距大于 limit 时，start 会被重设为 end - limit
* blockinfo 只包含很少量的信息

### /grpcpb.ApiService/GetSignedBlock

这个接口能获得一个块的详细信息

#### 请求

**GetSignedBlockRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| start | uint64 |  |  |

#### 返回

**GetSignedBlockResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| block | prototype.signed_block |  |  |

#### 代码示例

```go
func main() {
	conn, _ := grpc.Dial("localhost:8888",  grpc.WithInsecure())
	defer conn.Close()
	client := grpcpb.NewApiServiceClient(conn)
	req := &grpcpb.GetSignedBlockRequest{Start: 100}
	resp, _ := client.GetSignedBlock(context.Background(), req)
	fmt.Println(resp)
}
```

#### 说明

signed_block 中包括当前块全部交易。

### /grpcpb.ApiService/GetTrxListByTime

用来获取一段时间内的所有交易

#### 请求

**GetTrxListByTimeRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| start | prototype.time_point_sec |  |  |
| end | prototype.time_point_sec |  |  |
| limit | uint32 |  |  |
| last_info | TrxInfo |  |  |

#### 返回

**GetTrxListByTimeResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| list | TrxInfo | repeated |  |

#### 代码示例

```go
func main() {
	conn, _ := grpc.Dial("localhost:8888",  grpc.WithInsecure())
	defer conn.Close()
	client := grpcpb.NewApiServiceClient(conn)
	start := prototype.TimePointSec{UtcSeconds: 1567134897}
	end := prototype.TimePointSec{UtcSeconds: 1565134897}
	req := &grpcpb.GetTrxListByTimeRequest{Start: &start, End: &end, Limit: 50, LastInfo: nil}
	resp, _ := client.GetTrxListByTime(context.Background(), req)
	fmt.Println(resp)
}
```

#### 说明
1. 结果是倒序排列
2. start 大于 end，如果需要取 0 ~ 100 之间的数据，start 应该是 100，end 是 0
3. limit 最大为 100
4. 如果要实现遍历查询数据，last_info 是必要的，因为可能出现 end 时间戳上多个交易，并且被 limit 截断的情况。需要配合前一次查询的最后一条数据(last_info)查询。

### /grpcpb.ApiService/GetTrxInfoById

通过 trxid 获取一条交易的详情

#### 请求

**GetTrxInfoByIdRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| trx_id | prototype.sha256 |  |  |

#### 响应

**GetTrxInfoByIdResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| info | TrxInfo |  |  |

#### 代码示例

```go
func main() {
	conn, _ := grpc.Dial("localhost:8888",  grpc.WithInsecure())
	defer conn.Close()
	client := grpcpb.NewApiServiceClient(conn)
	hashStr := "0c46b086ca170e2526f35b9025683b904a280bcdf18a6335c43c3c46942e32c4"
	hash, _ := hex.DecodeString(hashStr)
	trxId := &prototype.Sha256{Hash: hash}
	req := &grpcpb.GetTrxInfoByIdRequest{TrxId: trxId}
	resp, _ := client.GetTrxInfoById(context.Background(), req)
	fmt.Println(resp)
}
```

### /grpcpb.ApiService/GetAccountByName

通过 account 的 name 查询链上 account 信息

#### 请求

**GetAccountByNameRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| account_name | prototype.account_name |  |  |

#### 响应

**AccountResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| info | AccountInfo |  |  |
| state | ChainState |  |  |

#### 代码示例

```go
func main() {
	conn, _ := grpc.Dial("localhost:8888",  grpc.WithInsecure())
	defer conn.Close()
	client := grpcpb.NewApiServiceClient(conn)
	req := &grpcpb.GetAccountByNameRequest{AccountName: &prototype.AccountName{Value: "initminer"}}
	resp , _ := client.GetAccountByName(context.Background(), req)
	fmt.Println(resp)
}
```

#### 说明
这个接口除了返回 account 的详细信息，还会返回链的全局变量

### /grpcpb.ApiService/GetPostListByCreateTime

查询一段时间以内发表的所有内容

#### 请求

**GetPostListByCreateTimeRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| start | prototype.time_point_sec |  |  |
| end | prototype.time_point_sec |  |  |
| last_post | PostResponse |  |  |
| limit | uint32 |  |  

#### 响应

**GetPostListByCreateTimeResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| posted_list | PostResponse | repeated |  |


#### 代码示例

```go
func main() {
	conn, _ := grpc.Dial("localhost:8888",  grpc.WithInsecure())
	defer conn.Close()
	client := grpcpb.NewApiServiceClient(conn)
	req := &grpcpb.GetPostListByCreateTimeRequest{Start: &prototype.TimePointSec{UtcSeconds: 1567493509},
		End: &prototype.TimePointSec{UtcSeconds: 1567483509}, LastPost:nil, Limit: 10}
	resp , _ := client.GetPostListByCreateTime(context.Background(), req)
	fmt.Println(resp)
}
```

#### 说明

1. 返回的 post 结果倒序排列
2. 因为倒序排列，start > end
3. limit 同样限制最大为 30
4. reply 也是一种 post，但是 parentId 不为 0

### /grpcpb.ApiService/GetPostInfoById

通过 postid 查询对应的 post

#### 请求

**GetPostInfoByIdRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| post_id | [uint64](#uint64) |  |  |
| voter_list_limit | [uint32](#uint32) |  |  |
| reply_list_limit | [uint32](#uint32) |  |  |


#### 响应

**GetPostInfoByIdResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| post_info | PostResponse |  |  |
| voter_list | VoterOfPost | repeated |  |
| reply_list | PostResponse | repeated |  |

#### 代码示例

```go
func main() {
	conn, _ := grpc.Dial("localhost:8888",  grpc.WithInsecure())
	defer conn.Close()
	client := grpcpb.NewApiServiceClient(conn)
	req := &grpcpb.GetPostInfoByIdRequest{PostId:1567493486154014778 }
	resp , _ := client.GetPostInfoById(context.Background(), req)
	fmt.Println(resp)
}
```

#### 说明

1. 这个接口不仅会返回对应的 post 信息，并且会返回这个 post 的 reply 与 vote 信息
2. 同样，返回的 reply 和 vote 最大个数也限制为 30

### /grpcpb.ApiService/BroadcastTrx

广播交易给 block producer 处理，唯一的写入接口

#### 请求

**BroadcastTrxRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| transaction | prototype.signed_transaction |  |  |
| only_deliver | bool |  |  |
| finality | bool |  |  |

#### 响应

**BroadcastTrxResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| invoice | prototype.transaction_receipt_with_info |  |  |
| status | uint32 |  |  |
| msg | string |  |  |
| finality | bool |  |  |

#### 代码示例

注意，所有链上写入操作都是通过这个接口。这个接口的核心是请求中携带的 transaction。transaction 是 signed_transaction 类型，需要签名。

**示例: 创建账户**

```go
func PublicKeyFromWIF(encoded string) *prototype.PublicKeyType {

	buffer := ([]byte(encoded))[3:]
	decoded, err := base58.BitcoinEncoding.Decode(buffer)
	if err != nil {
		return nil
	}

	x, ok := new(big.Int).SetString(string(decoded), 10)
	if !ok {
		return nil
	}

	buf := x.Bytes()
	if len(buf) <= 4 {
		return nil
	}

	temp := sha256.Sum256(buf[:len(buf)-4])
	temps := sha256.Sum256(temp[:])

	if !bytes.Equal(temps[0:4], buf[len(buf)-4:]) {
		return nil
	}

	result := new(prototype.PublicKeyType)
	result.Data = buf[:len(buf)-4]

	return result
}

func PrivateKeyFromWIF(encoded string) *prototype.PrivateKeyType {
	decoded, err := base58.BitcoinEncoding.Decode([]byte(encoded))
	if err != nil {
		return nil
	}

	x, ok := new(big.Int).SetString(string(decoded), 10)
	if !ok {
		return nil
	}

	buf := x.Bytes()
	if len(buf) <= 5 || buf[0] != 1 {
		return nil
	}
	buf = buf[1:]

	temp := sha256.Sum256(buf[:len(buf)-4])
	temps := sha256.Sum256(temp[:])

	if !bytes.Equal(temps[0:4], buf[len(buf)-4:]) {
		return nil
	}

	privateKey := new(prototype.PrivateKeyType)
	privateKey.Data = buf[:len(buf)-4]

	return privateKey
}

func Int2Bytes(n uint32) []byte {
	var b []byte
	var i int
	for i = 0; i < 4; i++ {
		b = append(b, 0)
	}
	i = 4
	for n > 0 && i > 0 {
		i--
		b[i] = byte(n & 0xff)
		n /= 256
	}
	return b
}

func GetTrxHash(trx *prototype.Transaction, cid prototype.ChainId) ([]byte, error) {
	buf, err := proto.Marshal(trx)

	if err != nil {
		return nil, err
	}

	h := sha256.New()

	cidBuf := Int2Bytes(cid.Value)
	h.Reset()
	h.Write(cidBuf)
	h.Write(buf)
	bs := h.Sum(nil)

	if bs == nil {
		return nil, errors.New("sha256 error")
	}

	return bs, nil
}

func GetChainIdByName(name string) uint32 {
	return crc32.ChecksumIEEE([]byte(name))
}

func Sign(signedTrx *prototype.SignedTransaction, secKey *prototype.PrivateKeyType, cid prototype.ChainId) []byte {
	buf, err := GetTrxHash(signedTrx.Trx, cid)

	if err != nil {
		return nil
	}

	res, err := secp256k1.Sign(buf, secKey.Data)

	if err != nil {
		return nil
	}

	return res
}

func GenerateSignedTxAndValidate(dgpo *prototype.DynamicProperties, op *prototype.AccountCreateOperation, privKey *prototype.PrivateKeyType) (*prototype.SignedTransaction, error) {

	const TaposMaxBlockCount = 0x800
	const Expire = 30

	refBlockPrefix := binary.BigEndian.Uint32(dgpo.HeadBlockId.Hash[8:12])
	refBlockNum := uint32(dgpo.HeadBlockNumber % TaposMaxBlockCount)


	tx := &prototype.Transaction{RefBlockNum: refBlockNum, RefBlockPrefix: refBlockPrefix, Expiration: &prototype.TimePointSec{UtcSeconds: dgpo.Time.UtcSeconds + Expire}}

	rawOp := new(prototype.Operation)
	opWrap := &prototype.Operation_Op1{Op1:op}
	rawOp.Op = opWrap
	tx.Operations = append(tx.Operations, rawOp)

	signTx := &prototype.SignedTransaction{Trx: tx}

	chainId := prototype.ChainId{Value: GetChainIdByName("main")}

	res := Sign(signTx, privKey, chainId)
	signTx.Signature = &prototype.SignatureType{Sig: res}

	return signTx, nil
}

func main() {
	conn, _ := grpc.Dial("localhost:8888",  grpc.WithInsecure())
	defer conn.Close()
	client := grpcpb.NewApiServiceClient(conn)
	fee := &prototype.Coin{Value: 1000000}
	newAccountPubkey := PublicKeyFromWIF("COS6cPDiXW4amg3dmMMQ9fqxMVJc7dtoJjaaYetBvX72q77yGQ9u1")
	creatorPrivkey := PrivateKeyFromWIF("4DjYx2KAGh1NP3dai7MZTLUBMMhMBPmwouKE8jhVSESywccpVZ")
	acop := &prototype.AccountCreateOperation{
		Fee:            fee,
		Creator:        &prototype.AccountName{Value: "initminer"},
		NewAccountName: &prototype.AccountName{Value: "testuser1"},
		PubKey:          newAccountPubkey,
	}
	req := &grpcpb.NonParamsRequest{}
	resp, _ := client.GetChainState(context.Background(), req)
	dgpo := resp.State.Dgpo
	signTx, _ := GenerateSignedTxAndValidate(dgpo, acop, creatorPrivkey)
	broadcastReq := &grpcpb.BroadcastTrxRequest{Transaction:signTx}
	broadcastResp, _ := client.BroadcastTrx(context.Background(), broadcastReq)
	fmt.Println(broadcastResp)
}
```

上述代码会正确运行，并在链上创建一个账号，账号名: testuser1，使用公钥 COS6cPDiXW4amg3dmMMQ9fqxMVJc7dtoJjaaYetBvX72q77yGQ9u1。
上述代码作为示例代码，没有对方法进行封装。GetTrxHash 和 Sign 方法作为 SignedTransaction 类的类方法无疑更加符合习惯。

#### 说明

由示例代码可以看出，broadcast 一个 transaction 非常复杂，并且涉及到了公私钥解析，签名等。

contentos 中最基本的操作是 Operation，包括创建账号，转账等。Transaction 包含了一个或者多个同一个人发起的 Operation，Transaction 被处理的时候，里面的 Operation 要么全部执行成功，要么全部执行失败。Transaction 的发起者对其用私钥签名之后，这个 Transaction 变成 SignedTransaction，后者在前者的基础上多了签名信息。签名生成的算法参考 Sign 函数。

上述代码中实现了 contentos 公私钥的解析函数，即 `PublicKeyFromWIF` 和 `PrivateKeyFromWIF` 生成算法请查阅源码。和大多数区块链项目类似， contentos 使用椭圆曲线生成公私钥，参数为 secp256k1。

contentos 不能随意创建账号，而是需要有账号的人进行创建。但是 initminer 这个账号是一定存在的，它在创世区块被自动创建。

AccountCreateOperaton 中需要如下：

1. 本次创建账号费用 Fee。这部分费用会从创建者账户的 balance 中扣除，并追加进被创建账户的 vest balance 中
2. 创建者的用户名 Creator
3. 被创建者的用户名 NewAccountName
4. 被创建者使用的公钥 PubKey

被创建者应该确保公钥对应的私钥不会泄漏。

contentos 通过 chainid 区分不同的链，默认的 chainid 是 main。

AccountCreateOperaton 被放入 Transaction 中，并由创建者使用自己的私钥进行签名。Transaction 需要当前链的 BlockPrefix 和 BlockNum 所以需要先查询一次链。

示例代码中 GenerateSignedTxAndValidate 实现只支持 AccountCreatorOperation ，这是为了简便和演示。更通用的代码请参考 contentos 源码中的同名函数。