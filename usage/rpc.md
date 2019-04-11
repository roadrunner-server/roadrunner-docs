# RPC Integration
You can connect to RoadRunner server from your PHP workers using shared RPC bus. In order to do that you have to create and an instance of `RPC` class configured to work with the address specified in `.rr` file.

```yaml
# rpc bus allows php application and external clients to talk to rr services.
rpc:
  # enable rpc server
  enable: true

  # rpc connection DSN. Supported TCP and Unix sockets.
  listen: tcp://127.0.0.1:6001
```

```php
$relay = new Spiral\Goridge\SocketRelay("127.0.0.1", 6001);
$rpc = new Goridge\RPC($relay);
```

You can immediately use this RPC to call embedded RPC services such as HTTP:

```php
var_dump($rpc->call('http.Workers', true));
```

> See RPC method definition [here](https://github.com/spiral/roadrunner/blob/master/service/http/rpc.go#L41).

You can read how to create your own services and RPC methods in [this section](Writing-Services).
