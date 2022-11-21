# Centrifuge plugin

Centrifuge plugin `[since 2.12.0]` comes to replace our `Websockets` and `Broadcast` deprecated plugins. 
Centrifuge is a robust library with a different clients (JS, Dart, Go, Python), which are actively supported.
Our previous JS client had very poor support and not updated for a long time.

RR exposes all available API including a `gRPC` proxy and client's API.

## API

####
- [PHP library](https://github.com/roadrunner-php/centrifugo)

#### Proxy interface

- [Docs](https://centrifugal.dev/docs/server/proxy#grpc-proxy)

RR follows the [proxy.proto](https://github.com/centrifugal/centrifugo/blob/master/internal/proxyproto/proxy.proto) specifications and proxies these events to the PHP worker.
To determine, inside the PHP what proxy method was called, RR adds an `type` : `endpoint` metadata. For example, if the `Subscribe` method was called, RR will add `type`:`subscribe` metadata to the worker's context.

#### RPC 

You may also use RPC methods to communicate with centrifugo server. RR follows the official [centrifugo proto API](https://github.com/centrifugal/centrifugo/blob/master/internal/apiproto/api.proto).
Official documentation available [here](https://centrifugal.dev/docs/server/server_api#grpc-api)