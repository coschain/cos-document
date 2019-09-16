# Configuration and terms

## You can know from this section

1. the configuration file
2. what is block producer
3. what is initminer

## Configuration file

Last section, we execuated `cosd init` which would generate default configuration folder `cosd` in `~/.coschain`. `cosd` contains a `config.toml`, generally it should not be modified.

The configuration file contains a number of sections and outmost include *ChainId* and *Name*.

Nodes can discover each other only when they have same *ChainId*. The default ChainId is *main* and can customize by `cosd init -c`. As same as multinodetester which default chainid is main and customize by flag `-c` like `multinodetester -c dev init 4`.

*Name* tell node which configuration file should be read. The default name of node is *cosd* that will read `~/.coschain/cosd`. As there were a node called testnode that would read `~/.coschain/testnode`.

### Consensu section

If *BootStrap* were set to *true*, when it was selected to be a block producer, would sign block by *LocalBpPrivateKey* as the name *LocalBpName*.

If *BootStrap* were set to *false*, the coschain would only sync blocks from others. Even it selected to be a block producer, it just skip this producing block turn.

### Database section

This section is optional, and would not be used by coschain. Only when user started some plugins, these field were be accepted. It is a standard data source configuration.

### GRPC section

*HTTPCors* is whitelist for [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).
*HTTPListen* is the port for coschain's reverse proxy which convert http request/response to grpc request/response and vice verse.
*RPCListen* is the port for coschain grpc service.

### HealthCheck section

The port for healthcheck

### P2P section

Recommand do not modify.

*SeedList* to set seed ip, recommand from offcial block producers.

## Terms

### Bp node

Coschain use SaBFT as its consensus protocol which is a type of BFT. Under this model, every time there is only one node have right to produce block and others in that time could just sync the block from the node. The node who can produce blocks are called block producer node aka. bp node. To be a block producer, the node owner have to hold enough vests, send a transaction to register itself as a bp. If successed, the node is called bp candidate.A candidate have to get enough votes to be a real bp node. As a bp, produce each block will be awarded.

### initminer

When coschain just start, the bp candidate list is empty. Initminer will set itself as a bp to produce blocks until there are other block producers selected. Initminer produces blocks won't be awarded.