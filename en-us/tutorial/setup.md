# Setup local environment

This tutorial begins from compiling source codes, and set up local environment step by step until the local chain can produce blocks. Considering the errors during compiling time, we also provide a [docker](). For a concrete knowledge about contentos, building from source code is alse a choice. 

## Step 1: Getting source code

Get source code from github

```
git clone git@github.com:coschain/contentos-go.git
```

## Step 2: Compling

### The golang version

Contentos requires [Go 1.11.4+](https://golang.org/dl/) and adopt officail [go modules](https://github.com/golang/go/wiki/Modules) to manage dependencies.

### Compiling cosd and wallet

All executable programs can be compiled are placed under /cmd/ like cosd and wallet. The former is coschain itself and latter is the tool to interact with coschain.

```shell
cd build
go build ../cmd/cosd
go build ../cmd/wallet-cli
```

can also add build flags like `go build -o`.

## Step 3: Running

### Initializing

The folder `~/.coschain` should be initialized to place configuration and database files before coschain running. Using command

```shell
cosd init
```

to create a default folder in `~/.coschain/cosd`.

You can also name the folder:

```shell
cosd init -n testnode
```

### Running

It is hard to run a coschain and let it work correctly in local environment. For quick beginning, we provide a shell script to do the job.

This section for who want to know the detail process about coschain's running.

#### Processing

##### Single node

Finished initializing, call `start` command to start coschain.

```shell
cosd start
``

Unless passed custom name to the command, this command would use the configuration file `config.toml` under `~/.coschain/cosd` to start the chain and write data file into the path.

```shell
cosd start -n testnode
```

Switch configuration file by using flag `-n`.

And then, one coschain node is running in local environment.

##### Multi nodes

Only one coschain node runing can not produce blocks. As BFT protocol, at lease three nodes are running and can communicate with each other, they could produce blocks. 

###### Using multinodetester

You can create multi node configuration in local manually, but to avoid trivial tasks like this, we provide multinodetester program to help.

It is also placed in /cmd/ and still need to be built.

```shell
cd build
go build ../cmd/multinodetester
```

**initialize 4 nodes**

```shell
multinodetester init 4
```

which creates testcosd_0 to testcosd_4 in  `~/.coschain` refers to 4 different nodes.

**running 4 nodes**

```shell
multinodetester start path/to/cosd 4
```

path/to/cosd refers to the path of execuable cosd program. It should be built before execuate this command.

After all above finished, there would be 4 nodes running in local environment.

##### Choosing a node to produce block

Even there are multi nodes running, but didn't choose a block producer, only `testcosd_0` (initminer) node would produces blocks and others just sync these produced blocks. So we need choose block producer manually. About the terms like initminer, block producer etc, read [Basic Concepts]().

In general, a node who wanted to be a block producer should first registered itself to block producer then persuade others to vote to it. In local environment, we simplified it and provide `multinodetester` command in wallet-cli to register and vote.

Prepare the `wallet-cli` which is a execuatable program that source code located in /cmd/ folder.

```shell
wallet-cli
```

Run the program and enter into interactive mode. Input `multinodetester N` to register and vote as block producers, N refers to the number of block producers. In there, `multinodetester 4` is ok.

Noninteractive mode is provided. Run wallet-cli directly as below.

```shell
wallet-cli multinodetester 4
```

And now, the coschain is running in local and can produce blocks, try to [interactive with chain](), but before that, some terms and concepts should be explained.