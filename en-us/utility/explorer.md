# explorer

## Intro

*explorer* is a blockchain explorer for contentos, to show the running information or statistics of current chain. The testnet's explorer was online，
visit <http://explorer.contentos.io> to know more.

## Local Explorer

Using explorer to monitor local chain by two ways:

### customize rpc

<http://explorer.contentos.io> can `change rpc`. After a contentos chain was running in local, you can change rpc to `http://0.0.0.0:8080` to see local status.

### building in local

Get the source code from [explorer](https://github.com/coschain/block-explorer)

Require `nodejs` >= `4.0.0` and `npm` >= `3.0.0`.

Running `npm install && npm run dev`. By default, explorer will read data from <http://localhost:8080> if cosd is running well in local. 
Visit <http://localhost:9091> to see the information.
