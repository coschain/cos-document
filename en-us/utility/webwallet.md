# web-wallet

## intro

`web-wallet` is a web version of `wallet-cli` from contentos-go. It is a subset and only supports below commands:

* create account
* generate keystore file
* transfer
* post

Visit the web-wallet for testnet in <https://testwallet.contentos.io/>.

## Building

Get source from [cos-web-toolkit](https://github.com/coschain/cos-web-toolkit)  

Require `nodejs` >= `6.0.0` and `npm` >= `3.0.0`.

Running `npm install` to install dependencies.

## Running

### dev environment
`web-wallet` composed by frontend and backend. You can run frontend by `npm run dev` and `npm run debug_server` to run backend.

Read detail from `package.json`, `debug_server` need some environment variables especially `INITMINER`. 
Which refers to the private key belongs to initminer in contentos-chain. If you had modified it in `contentos-go`, you should do it in `package.json` as well.

Using the default private key is dangerous. Do modify it before you release your own contentos-chain.

### under pm2
The default configuration file is `pm2.config.js`. Watch out the `error_file` and `out_file` should be modified.

After `npm run build`, you can run web-wallet by `INITMINER=yourPrivKey pm2 start pm2.config.js --env development`. 
The `INITMINER` should be modified to your own private key. 

