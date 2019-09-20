# RPC to App Server
You can connect to application server via `SocketRelay`:

```php
$relay = new Spiral\Goridge\SocketRelay("127.0.0.1", 6001);
$rpc = new Spiral\Goridge\RPC($relay);
```

You can immediately use this RPC to call embedded RPC services such as HTTP:

```php
var_dump($rpc->call('http.Workers', true));
```

> See RPC method definition [here](https://github.com/spiral/roadrunner/blob/master/service/http/rpc.go#L41).

You can read how to create your own services and RPC methods in [this section](/beep-beep/service.md).
