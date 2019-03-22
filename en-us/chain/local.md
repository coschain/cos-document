# Local Environment

## Get Source Codes

Contentos is a open-source project, you can visit [contentos-go](https://github.com/coschain/contentos-go) on github and 
get latest source code using any git-like tool like [git](https://git-scm.com/) from master branch or only 
[release](https://github.com/coschain/contentos-go/releases) versions.

Be carefully, Although the main chain was almost released, which is still under heavy development. Breaking changes are actively added.

## Dependency

Requires [Go 1.11.4+](https://golang.org/dl/)

## Building

There are two executable program can be built from source.

* cosd: the main chain service 
* wallet-cli: the client to interact with chain 

You can 
```bash
make build_cosd
```

And

```bash
make build_wallet
```

to build both.

### wallet-cli

Run the program and it will enter 'interactive mode'. Make sure the `cosd` has been running in local environment. It supports varies commands

* account
* bp
* claim
* close
* create
* follow
* genKeyPair
* import
* info
* locked
* list
* load
* lock
* post
* reply
* transfer
* unlock
* transfer_vesting
* vote

Input `--help` to query detail information of command.

We released a web-wallet as well, to visit [cos-web-toolkit](https://github.com/coschain/cos-web-toolkit). 
Or you can read [web-wallet](../utility/webwallet.md) to know more.

### cosd

Run `cosd init` will initialize local environment for contentos. The default configuration located in `$HOME/.coschain`.
After being initialized, input `cosd start` to make the chain running in local. By default, 
the cosd tries to open ports including `8080`, `8888` and `20338`. Do not occupied these ports.

### warning
The consensus of contentos restricts at lease there are three different nodes it could process normally. 
If there is only one node running in local, after 1024 seconds, it would exit immediately.



