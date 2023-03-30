# RPC to RoadRunner

You can connect to application server via `SocketRelay`:

```php
$rpc = \Spiral\Goridge\RPC\RPC::create('tcp://127.0.0.1:6001');
```

You can immediately use this RPC to call embedded RPC services:

```php
var_dump($rpc->call('informer.Workers', 'http'));
```

> **INFO**
> Please note that in the case of running workers in debug mode:
> `http: { pool.debug: true }` (in `.rr.yaml`) the number of http workers will be zero (i.e. an empty array `[]` will be
> returned).
>
> This behavior may be changed in the future, you should not rely on this result to check that the
> RoadRunner was launched in development mode.

You can read how to create your own services and RPC methods in [this section](../customization/plugin.md).

You can connect to RoadRunner server from your PHP workers using shared RPC bus. In order to do that you have to create
an instance of `RPC` class configured to work with the address specified in `.rr` file.

## Requirements

To connect to RoadRunner from PHP application in RPC mode you need:

- ext-sockets
- ext-json

## Configuration

To change the RPC port from the default (localhost:6001) use:

```yaml
version: "3"

rpc:
  listen: tcp://127.0.0.1:6001
```

```php
$rpc = Goridge\RPC\RPC::create(RoadRunner\Environment::fromGlobals()->getRPCAddress());
```

You can immediately use this RPC to call embedded RPC services such as HTTP:

```php
var_dump($rpc->call('informer.Workers', 'http'));
```

## Go RPC

You may be able to find the RPC Go definitions here:

- [Jobs](https://github.com/roadrunner-server/jobs/blob/master/rpc.go)
- [KV](https://github.com/roadrunner-server/kv/blob/master/rpc.go)
- [Informer](https://github.com/roadrunner-server/informer/blob/master/rpc.go)
- [Resetter](https://github.com/roadrunner-server/resetter/blob/master/rpc.go)
- [Status](https://github.com/roadrunner-server/status/blob/master/rpc.go)
- [Metrics](https://github.com/roadrunner-server/metrics/blob/master/rpc.go)
- [Lock](https://github.com/roadrunner-server/lock/blob/master/rpc.go)
