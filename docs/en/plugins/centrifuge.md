# Plugins — Centrifuge

The RoadRunner Centrifuge plugin provides seamless integration with [Centrifugo](https://centrifugal.dev/), a powerful
websocket server. This plugin allows you to proxy events from Websocket serer to PHP workers running on RoadRunner and
send data back to the websocket client, enabling real-time communication between the server and the client.

The plugin provides the following features:

1. **Event Proxying:** The plugin allows RoadRunner to receive all the events from Centrifugo server such
   as [Connect](https://centrifugal.dev/docs/server/proxy#connect-proxy),
   [Refresh](https://centrifugal.dev/docs/server/proxy#refresh-proxy),
   [RPC](https://centrifugal.dev/docs/server/proxy#rpc-proxy),
   [Subscribe](https://centrifugal.dev/docs/server/proxy#subscribe-proxy),
   [Publish](https://centrifugal.dev/docs/server/proxy#publish-proxy),
   and [Sub refresh](https://centrifugal.dev/docs/server/proxy#sub-refresh-proxy). It then proxies these events into PHP
   workers and sends a result from PHP application back to the websocket client.
2. **RPC API**: The plugin provides an RPC API that allows you to send data to the websocket server. For example, you
   can publish or broadcast data into a channel from PHP application using the API.

> **Note**
> The Centrifuge plugin `[since 2.12.0]` replaces our deprecated `Websockets` and `Broadcast` plugins.

In addition, Centrifugo provides a convenient [JavaScript library](https://centrifugal.dev/docs/transports/client_api)
that simplifies the development of real-time applications.

## PHP client

The RoadRunner centrifuge plugin comes with a convenient PHP package that simplifies the process of integrating the
plugin with your PHP application. The package provides a set of classes and functions that handle incoming events from
Centrifugo and allow you to send data back to the websocket server via RPC.

### Installation

To install the package, run the following command:

```terminal
composer require roadrunner-php/centrifugo
```

### Configuration

First you need to add centrifuge section to your RoadRunner configuration. For example, such a configuration would be
quite feasible to run:

```yaml .rr.yaml
rpc:
  listen: tcp://127.0.0.1:6001

server:
  command: "php app.php"
  relay: pipes

centrifuge:
  # Centrifugo server proxy address (docs: https://centrifugal.dev/docs/server/proxy#grpc-proxy)
  # Optional, default: tcp://127.0.0.1:30000
  proxy_address: "tcp://127.0.0.1:30000"

  # gRPC server API address (docs: https://centrifugal.dev/docs/server/server_api#grpc-api)
  # Optional, default: tcp://127.0.0.1:30000. Centrifugo: `grpc_api` should be set to true and `grpc_port` should be the same as in the RR's config.
  grpc_api_address: tcp://127.0.0.1:30000

  # Use gRPC gzip compressor
  # Optional, default: false
  use_compressor: true

  # Your application version
  # Optional, default: v1.0.0
  version: "v1.0.0"

  # Your application name
  # Optional, default: roadrunner
  name: "roadrunner"

  # TLS configuration
  # Optional, default: null
  tls:
    # TLS key
    # Required
    key: /path/to/key.pem

    # TLS certificate
    # Required
    cert: /path/to/cert.pem
```

And also you need to configure Centrifugo server to use RoadRunner as a proxy.

For example:

```json
{
  "admin": true,
  "api_key": "secret",
  "admin_password": "password",
  "admin_secret": "admin_secret",
  "allowed_origins": [
    "*"
  ],
  "token_hmac_secret_key": "test",
  "proxy_publish": true,
  "proxy_subscribe": true,
  "allow_subscribe_for_client": true,
  "proxy_connect_endpoint": "grpc://127.0.0.1:10001",
  "proxy_connect_timeout": "10s",
  "proxy_publish_endpoint": "grpc://127.0.0.1:10001",
  "proxy_publish_timeout": "10s",
  "proxy_subscribe_endpoint": "grpc://127.0.0.1:10001",
  "proxy_subscribe_timeout": "10s",
  "proxy_refresh_endpoint": "grpc://127.0.0.1:10001",
  "proxy_refresh_timeout": "10s",
  "proxy_sub_refresh_endpoint": "grpc://127.0.0.1:10001",
  "proxy_sub_refresh_timeout": "1s",
  "proxy_rpc_endpoint": "grpc://127.0.0.1:10001",
  "proxy_rpc_timeout": "10s"
}
```

> **Note**
> `proxy_connect_endpoint`, `proxy_publish_endpoint`, `proxy_subscribe_endpoint`, `proxy_refresh_endpoint`, `proxy_sub_refresh_endpoint`, `proxy_rpc_endpoint` -
> endpoint address of roadrunner server with activated centrifuge plugin.

### PHP worker example

Here is an example of a PHP worker:

```php centrifuge-worker.php
<?php

require __DIR__ . '/vendor/autoload.php';

use RoadRunner\Centrifugo\CentrifugoWorker;
use RoadRunner\Centrifugo\Payload;
use RoadRunner\Centrifugo\Request;
use RoadRunner\Centrifugo\Request\RequestFactory;
use Spiral\RoadRunner\Worker;

$worker = Worker::create();
$requestFactory = new RequestFactory($worker);

// Create a new Centrifugo Worker from global environment
$centrifugoWorker = new CentrifugoWorker($worker, $requestFactory);

while ($request = $centrifugoWorker->waitRequest()) {

    if ($request instanceof Request\Invalid) {
        $errorMessage = $request->getException()->getMessage();

        if ($request->getException() instanceof \RoadRunner\Centrifugo\Exception\InvalidRequestTypeException) {
            $payload = $request->getException()->payload;
        }

        // Handle invalid request
        // $logger->error($errorMessage, $payload ?? []);

        continue;
    }

    if ($request instanceof Request\Refresh) {
        try {
            // Do something
            $request->respond(new Payload\RefreshResponse(
                // ...
            ));
        } catch (\Throwable $e) {
            $request->error($e->getCode(), $e->getMessage());
        }

        continue;
    }

    if ($request instanceof Request\Subscribe) {
        try {
            // Do something
            $request->respond(new Payload\SubscribeResponse(
                // ...
            ));

            // You can also disconnect connection
            $request->disconnect('500', 'Connection is not allowed.');
        } catch (\Throwable $e) {
            $request->error($e->getCode(), $e->getMessage());
        }

        continue;
    }

    if ($request instanceof Request\Publish) {
        try {
            // Do something
            $request->respond(new Payload\PublishResponse(
                // ...
            ));

            // You can also disconnect connection
            $request->disconnect('500', 'Connection is not allowed.');
        } catch (\Throwable $e) {
            $request->error($e->getCode(), $e->getMessage());
        }

        continue;
    }

    if ($request instanceof Request\RPC) {
        try {
            $response = $router->handle(
                new Request(uri: $request->method, data: $request->data),
            ); // ['user' => ['id' => 1, 'username' => 'john_smith']]

            $request->respond(new Payload\RPCResponse(
                data: $response
            ));
        } catch (\Throwable $e) {
            $request->error($e->getCode(), $e->getMessage());
        }

        continue;
    }
}
```

## Protobuf API

To make it easy to use the Centrifugo proto API in PHP, we provide
a [GitHub repository](https://github.com/roadrunner-php/roadrunner-api-dto), that contains all the generated
PHP DTO classes for the Centrifugo proxy and API proto files, making it easy to work with these files in your PHP
application.

### Proxy interface

- [Docs](https://centrifugal.dev/docs/server/proxy#grpc-proxy)

RR follows the [proxy.proto](https://github.com/centrifugal/centrifugo/blob/master/internal/proxyproto/proxy.proto)
specifications and proxies these events to the PHP worker.
To determine what proxy method was called inside the PHP, RR adds a `type` : `endpoint` metadata. For example, if
the `Subscribe` method was called, RR will add `type`:`subscribe` metadata to the worker's context.

### RPC

You may also use RPC methods to communicate with centrifugo server. RR follows the
official [centrifugo proto API](https://github.com/centrifugal/centrifugo/blob/master/internal/apiproto/api.proto).
Official documentation available [here](https://centrifugal.dev/docs/server/server_api#grpc-api)

## Metrics

RoadRunner has a [metrics plugin](../lab/metrics.md) that provides metrics for the Centrifuge plugin, which can be used
with Prometheus.

![centrifuge-metrics](https://user-images.githubusercontent.com/773481/235842147-5f39a812-c67e-4b96-8dc6-dc2d61ceee3b.png)
