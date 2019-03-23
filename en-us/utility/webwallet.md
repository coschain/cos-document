# web-wallet

## intro

`web-wallet` is a web version of `wallet-cli` from contentos-go. It is a subset and only supports below commands:

* create account
* generate keystore file
* transfer
* post
* faucet (only supported on testnet)

Faucet drips One COS each time to make developers be able to interact with testnet.

Visit the web-wallet for testnet in <https://testwallet.contentos.io/>.

## Building

Get source from [cos-web-toolkit](https://github.com/coschain/cos-web-toolkit)  

Require `nodejs` >= `6.0.0` and `npm` >= `3.0.0`.

Running `npm install` to install dependencies.

## Running

### dev environment
`web-wallet` composed by frontend and backend. You can run frontend by `npm run dev` and `npm run debug_server` to run backend.

Read detail from `package.json`, `debug_server` need some environment variables especially `CREATOR`. 
Which refers to the private key belongs to an account called **accountcreator** in contentos-chain. 
The accountcreator is a special account which created by initminer and to create other accounts by web request.
Different from initminer, the accountcreator **would not** be created automatically after the chain's running. You should create it by yourself for local environment. 

Using the default private key is dangerous. Do modify it before you release your own contentos-chain.

### under pm2
The default configuration file is `pm2.config.js`. Watch out the `error_file` and `out_file` should be modified.

After `npm run build`, you can run web-wallet by `CREATOR=yourPrivKey pm2 start pm2.config.js --env development`. 
The `CREATOR` should be modified to your own private key. 

