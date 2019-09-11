# Chain Api

## Overview

CosChain provides public APIs by [gRPC](https://grpc.io/).
CosChain opened port 8888 for gRPC, and 8080 for gRPC-web. The latter works on http protocol. If coschain received a http request from port 8080, it converted the request to grpc stream and wrapped grpc response to http response then sent it back through the port. 
Read [grpc-web](https://github.com/improbable-eng/grpc-web) to learn more.
We provide an [online wallet](https://github.com/coschain/cos-web-toolkit) bases on grpc-web.

## Environment

Different from these APIs were built on http, before calling a gRPC API the structures of request and response should be known. You can find whole gRPC's defination on [grpc.proto](https://github.com/coschain/contentos-go/blob/master/rpc/pb/grpc.proto).

Grpc.proto depends on some type defination files: [type.proto](https://github.com/coschain/contentos-go/blob/master/prototype/type.proto), [multi_id.proto](https://github.com/coschain/contentos-go/blob/master/prototype/multi_id.proto), [operation.proto](https://github.com/coschain/contentos-go/blob/master/prototype/operation.proto), [transaction.proto](https://github.com/coschain/contentos-go/blob/master/prototype/transaction.proto).

Collecting all '.proto' files and compiling then you can request data from coschain.

## Convention

1. A gRPC API below will be presented as /grpcpb.ApiService/ + grpc method name. For Example, **/grpcpb.ApiService/GetChainState** refers to **rpc GetChainState (NonParamsRequest) returns (GetChainStateResponse)** in **grpc.proto**.
2. Using golang grammer.
3. Some type need add helper functions, looking up the helper functions defined in [prototype](https://github.com/coschain/contentos-go/tree/master/prototype).

## Apis

### /grpcpb.ApiService/GetChainState

This interface can query current state or chain global variables from coschain.

#### Request

NonParams

#### Response

**GetChainStateResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| last_irreversible_block_number | uint64 |  |  |
| last_irreversible_block_time | uint64 |  |  |
| dgpo | prototype.dynamic_properties |  |  |

#### Example

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

This interface can fetch blocks between two block heights.

#### Request

**GetBlockListRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| start | uint64 |  |  |
| end | uint64 |  |  |
| limit | uint32 |  |  |

#### Response

**GetBlockListResponse**


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| blocks | BlockInfo | repeated |  |

#### Example

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

#### Tips

* end can be set to 0 which refers to current latest block.
* limit is limited to 100
* if the distance from start to end greater than limit, the start will automatic reset to end - limit
* blockinfo only contain few information

### /grpcpb.ApiService/GetSignedBlock

Read block detail infomation

#### Request

**GetSignedBlockRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| start | uint64 |  |  |

#### Response

**GetSignedBlockResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| block | prototype.signed_block |  |  |

#### Example

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

#### Tips

signed_block contains all transactions belongs to the block.

### /grpcpb.ApiService/GetTrxListByTime

Get all transactions during a period of time.

#### Request

**GetTrxListByTimeRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| start | prototype.time_point_sec |  |  |
| end | prototype.time_point_sec |  |  |
| limit | uint32 |  |  |
| last_info | TrxInfo |  |  |

#### Response

**GetTrxListByTimeResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| list | TrxInfo | repeated |  |

#### Example

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

#### Tips

1. The returned transactions are in reversed ordere.
2. start should greater than end.
3. limit should less than 100
4. If wanted to fetch transactions by page, the last_info is useful.

### /grpcpb.ApiService/GetTrxInfoById

Get detail info in a transaction.

#### Request

**GetTrxInfoByIdRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| trx_id | prototype.sha256 |  |  |

#### Response

**GetTrxInfoByIdResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| info | TrxInfo |  |  |

#### Example

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

Get account info by a name

#### Request

**GetAccountByNameRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| account_name | prototype.account_name |  |  |

#### response

**AccountResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| info | AccountInfo |  |  |
| state | ChainState |  |  |

#### Example

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

#### Tips

Besides detail account info, the interface also returns chain state.

### /grpcpb.ApiService/GetPostListByCreateTime

Get All posts between a period.

#### Request

**GetPostListByCreateTimeRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| start | prototype.time_point_sec |  |  |
| end | prototype.time_point_sec |  |  |
| last_post | PostResponse |  |  |
| limit | uint32 |  |  

#### Response

**GetPostListByCreateTimeResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| posted_list | PostResponse | repeated |  |


#### Example

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

#### Tips

1. The posts response ordered by reversed.
2. start should be greater than end.
3. limit should be less than 30.
4. reply is also a post but parentId is not 0

### /grpcpb.ApiService/GetPostInfoById

Get post info by post id

#### Request

**GetPostInfoByIdRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| post_id | [uint64](#uint64) |  |  |
| voter_list_limit | [uint32](#uint32) |  |  |
| reply_list_limit | [uint32](#uint32) |  |  |


#### Response

**GetPostInfoByIdResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| post_info | PostResponse |  |  |
| voter_list | VoterOfPost | repeated |  |
| reply_list | PostResponse | repeated |  |

#### Example

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

#### Tips

1. Not only post infomation, the interface also returns replies and votes info.
2. the number of replies and votes are limited to 30

### /grpcpb.ApiService/GetBlockProducerByName

get block producer detail info by name

#### Request

**GetBlockProducerByNameRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| bp_name | prototype.account_name |  |  |

#### Response

**BlockProducerResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| owner | prototype.account_name |  |  |
| created_time | prototype.time_point_sec |  |  |
| url | string |  |  |
| bp_vest | prototype.bp_vest_id |  |  |
| signing_key | prototype.public_key_type |  |  |
| proposed_stamina_free | uint64 |  |  |
| tps_expected | uint64 |  |  |
| account_create_fee | prototype.coin |  |  |
| top_n_acquire_free_token | uint32 |  |  |
| ticket_flush_interval | uint64 |  |  |
| per_ticket_price | prototype.coin |  |  |
| per_ticket_weight | uint64 |  |  |
| voter_count | uint64 |  |  |
| gen_block_count | uint64 |  |  |


#### Example

```go
func main() {
	conn, _ := grpc.Dial("localhost:8888",  grpc.WithInsecure())
	defer conn.Close()
	client := grpcpb.NewApiServiceClient(conn)
	req := &grpcpb.GetBlockProducerByNameRequest{BpName: &prototype.AccountName{Value: "initminer1"}}
	resp , _ := client.GetBlockProducerByName(context.Background(), req)
	fmt.Println(resp)
}
```

### /grpcpb.ApiService/GetBlockProducerListByVoteCount

Get block producers list ordered by reversed votecount.

#### Request

**GetBlockProducerListByVoteCountRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| start | prototype.vest |  |  |
| end | prototype.vest |  |  |
| last_block_producer | BlockProducerResponse |  |  |
| limit | uint32 |  |  |

#### Response

**GetBlockProducerListResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| block_producer_list | BlockProducerResponse | repeated |  |

#### Example

```go
func main() {
	conn, _ := grpc.Dial("localhost:8888",  grpc.WithInsecure())
	defer conn.Close()
	client := grpcpb.NewApiServiceClient(conn)
	req := &grpcpb.GetBlockProducerListByVoteCountRequest{Limit: 10}
	resp , _ := client.GetBlockProducerListByVoteCount(context.Background(), req)
	fmt.Println(resp)
	// paginate
	bp := resp.BlockProducerList[0]
	start := bp.BpVest.VoteVest
	req = &grpcpb.GetBlockProducerListByVoteCountRequest{Start: start, LastBlockProducer: bp, Limit: 10}
	resp , _ = client.GetBlockProducerListByVoteCount(context.Background(), req)
	fmt.Println(resp)
}
```

#### Tips

limit should be less than 30

### /grpcpb.ApiService/BroadcastTrx

Commit and broadcast an executable transaction to one of coschain-nodes.

#### Request

**BroadcastTrxRequest**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| transaction | prototype.signed_transaction |  |  |
| only_deliver | bool |  |  |
| finality | bool |  |  |

#### Response

**BroadcastTrxResponse**

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| invoice | prototype.transaction_receipt_with_info |  |  |
| status | uint32 |  |  |
| msg | string |  |  |
| finality | bool |  |  |

#### Example

**Example: create an account**

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

Execuating above snippet will create a new account on local coschain. The new account uses testuser1 as its name with pubkey COS6cPDiXW4amg3dmMMQ9fqxMVJc7dtoJjaaYetBvX72q77yGQ9u1.
The snippet flattens execution processing to make it easy to read. In real life, the methods should be encapsulated. You can read related implement in [prototype](https://github.com/coschain/contentos-go/tree/master/prototype).

#### Explain

As the snippet, broadcast a transaction is a hard work which is related to many aspects include how to parse a public/private key or how to signed a message etc.

The transaction in BroadcastTrxRequest is signed_transaction type which combines a normal transaction type with a signature, and each transaction contains at least one operations. In coschain, operation is the elementary execution unit including create_account_operation, transfer_operation and account_update_operation etc.

All types of Operation are defined in [operation.proto](https://github.com/coschain/contentos-go/blob/master/prototype/operation.proto).

PubKey/PrivakeKey can be presented as string for human readable. To decode it to raw pubkey/privatekey type, you can read both `PublicKeyFromWIF` and `PrivateKeyFromWIF` functions above. Contentos' pubkey/privatekey generating algorithm is closed to others projects like btc or eth bases on ecdsa with secp256k1.

Detail execution processing of signature generating can read above Sign function.

You can't create account except you knew a private key belongs to other account who already have been recorded in coschain. For local environment or development environment you could use initminer as any other accounts' creator who was automaticly created in genesis block by coschain itself.

The arguments of AccountCreateOperation

1. Fee: substrct from creator's balance and add to createe's vest balance.
2. Creator: who create the createe
3. NewAccountName: the createe's name
4. PubKey: the createe's public key.

Createe should keep the private key and avoid to be leaked.

Different chainid refer to different chain, and cannot communicate with each other.

Before sign the transaction contains AccountCreateOperaton, the current blockprefix and blocknum should be known, query current chain state is necessary.

In sample snippet, GenerateSignedTxAndValidate function only support AccountCreatorOperation. For usual implememt, looking [GenerateSignedTxAndValidate](https://github.com/coschain/contentos-go/blob/master/cmd/wallet-cli/commands/utils/utils.go#L24).