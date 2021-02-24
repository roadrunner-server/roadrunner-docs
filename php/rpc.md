# RPC to App Server
You can connect to application server via `SocketRelay`:

```php
$rpc = \Spiral\Goridge\RPC\RPC::create('tcp://127.0.0.1:6001');
```

You can immediately use this RPC to call embedded RPC services such as HTTP:

```php
var_dump($rpc->call('informer.Workers', 'http'));
```

You can read how to create your own services and RPC methods in [this section](/beep-beep/plugin.md).
