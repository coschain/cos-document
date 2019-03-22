# js-sdk

## intro

The js sdk for contentos. Both [web-wallet](webwallet.md) and [explorer](webwallet.md) interact with cosd though it.

You can install it by `npm install cos-grpc-js` or build from source in [cos-js-sdk](https://github.com/coschain/cos-sdk-grpc-js) 

## usage

sdk basis on [grpc-web](https://github.com/improbable-eng/grpc-web). 

Do not modify any files in `src/lib/prototype` and `src/lib/grpc` which generated from contentos-go. 

Read [grpc-web](https://github.com/improbable-eng/grpc-web) and [cos-web-toolkit](https://github.com/coschain/cos-web-toolkit/blob/master/src/encrypt/clientsign.js) to 
know how to use it.
