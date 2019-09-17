# Plugins

## Overview

Plugins are optional service, provide extra data or functions. They need be invoked when cosd starting, and mounted on cosd. In defalut, they are inactive. Some services depend on others.

Mostly, plugins depend on standard database like [mysql](https://www.mysql.com/) and recommend version is 5.8+. Modify configuration file in ~/.coschain to make data source correct.

## Current plugins

### trxsqlservice

Write contentos transactions into database

#### Start

```bash
./cosd --plugin=trxsqlservice
```

#### Tips

Initialize database tables by `cosd db init` before cosd start

### block_log

Subscribe states changing. Different from trxsqlservice, it can be used to observe dynamic data like balance's changing because a contract calling. Also, block_log observes some global variables like per block's reward.

block_log observes properties below

1. create a account
2. economy
3. balance changing include COS and VEST
4. stake COS to STAKE
5. vote to block producer
6. transfer

Different data is written to different tables.

#### Start

```bash
./cosd --plugin=block_log_svc --plugin=block_log_proc_svc
```

### dailystatservice

Provide dapp info, only when the service started, grpc interface */grpcpb.ApiService/GetDailyStats* can work.

#### Start

```bash
./cosd --plugin=dailystatservice
```

#### Tips

dailystatservice service depends on trxsqlservice and block_log. In general, it should be running as:

```bash
./cosd --plugin=block_log_svc --plugin=block_log_proc_svc --plugin=trxsqlservice --plugin=dailystatservice
```