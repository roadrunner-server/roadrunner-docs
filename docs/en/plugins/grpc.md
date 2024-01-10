# Plugins — gRPC

RoadRunner gRPC plugin enables PHP applications to communicate with gRPC clients.

It consists of two main parts:

1. **protoc-plugin `protoc-gen-php-grpc`:** This is a plugin for the protoc compiler that generates PHP code from a gRPC
   service definition file (`.proto`). It generates PHP classes that correspond to the service definition and message
   types. These classes provide an interface for handling incoming gRPC requests and sending responses back to the
   client.
2. **gRPC server:** This is a server that starts PHP workers and listens for incoming gRPC requests. It receives 
   requests from gRPC clients, proxies them to the PHP workers, and sends the responses back to the client. The server 
   is responsible for managing the lifecycle of the PHP workers and ensuring that they are available to handle requests.

## Protoc-plugin

The first step is to define a `.proto` file that describes the gRPC service and messages that your PHP application will
handle.

In our documentation, we will use the following example of a `.proto` file that is stored in the `<app>/proto`
directory:

```proto proto/pinger.proto
syntax = "proto3";

option php_namespace = "GRPC\\Pinger";
option php_metadata_namespace = "GRPC\\GPBMetadata";

package pinger;

service Pinger {
  rpc ping (PingRequest) returns (PingResponse) {}
}

message PingRequest {
  string url = 1;
}

message PingResponse {
  int32 status_code = 1;
}
```

It defines a simple gRPC service called `Pinger` that takes a URL as input and returns the HTTP status code for that
URL.

The `php_namespace` and `php_metadata_namespace` options allow you to specify the namespaces to use in the generated DTO
and Service interface.

### Generating PHP code

After defining the proto file, you need to generate the PHP files using the `protoc` compiler and
the `protoc-gen-php-grpc` plugin. You can install the plugin binary using Composer or download a pre-built binary from
the GitHub releases page.

:::: tabs

::: tab Pre-built Binary

The simplest way to get the latest version of `protoc-gen-php-grpc` plugin is to download one of the pre-built release
binaries on the GitHub [releases page](https://github.com/roadrunner-server/roadrunner/releases).

Just download the appropriate archive from the release page and extract it into your desired application directory.

:::

::: tab Composer

If you use Composer to manage your PHP dependencies, you can install the `spiral/roadrunner-cli` package to download the
latest version of `protoc-gen-php-grpc` plugin to your project's root directory.

**Install the package**

```terminal
composer require spiral/roadrunner-cli
```

And run the following command to download the latest version of the plugin

```terminal
./vendor/bin/rr download-protoc-binary
```

Server binary will be available at the root of your project.

> **Warning**
> PHP's extensions `php-curl` and `php-zip` are required. Check with `php --modules` your installed extensions.

:::

::::

Once the plugin is installed, you can use the `protoc` command to compile the proto file into PHP files.

**Here's an example command:**

```bash
protoc --plugin=protoc-gen-php-grpc \
       --php_out=./generated \
       --php-grpc_out=./generated \
       proto/pinger.proto
```

> **Note**
> Make sure that the `generated` directory exists and is writable.

After running the command, you can find the generated DTO and `PingerInterface` files in
the `<app>/generated/GRPC/Pinger` directory.

We recommend also registering the GRPC namespace in the `composer.json` file:

```json
{
  "autoload": {
    "psr-4": {
      ...
      "GRPC\\": "generated/GRPC"
    }
  }
}
```

By doing this, you can easily use the generated PHP classes in your application.

### Using BUF plugin

You can also use the [BUF](https://buf.build/community/roadrunner-server-php-grpc) plugin
to generate PHP code from the `.proto` file.

## PHP Client

The RoadRunner gRPC plugin comes with a convenient PHP package that simplifies the process of integrating the plugin
with your PHP application.

### Installation

You can install the package via Composer using the following command:

```terminal
composer require spiral/roadrunner-grpc
```

### Implement Service

Next, you will need to create a PHP class that implements the `Pinger` service defined in the `.proto` file. This class
should implement the `GRPC/Pinger/PingerInterface`.

Here's an example:

```php
use Spiral\RoadRunner\GRPC;
use GRPC\Pinger\PingerInterface;
use GRPC\Pinger\PingRequest;
use GRPC\Pinger\PingResponse;

final class Pinger implements PingerInterface
{
    public function __construct(
        private readonly HttpClientInterface $httpClient
    ) {
    }
    
    public function ping(GRPC\ContextInterface $ctx, PingRequest $in): PingResponse
    {
        $statusCode = $this->httpClient->get($in->getUrl())->getStatusCode();
    
        return new PingResponse([
            'status_code' => $statusCode
        ]);
    }
}
```

### Usage

To use the `Pinger` service, you can create a PHP worker that registers the service with the gRPC server.

Here's an example of how to do this:

```php grpc-worker.php
use GRPC\Pinger\PingerInterface;
use Spiral\RoadRunner\GRPC\Server;
use Spiral\RoadRunner\Worker;

require __DIR__ . '/vendor/autoload.php';

$server = new Server(null, [
    'debug' => false, // optional (default: false)
]);

$server->registerService(PingerInterface::class, new Pinger(new HttpClient()));

$server->serve(Worker::create());
```

After creating the worker, you need to configure RoadRunner to register the `proto/pinger.proto` service.

Here's an example configuration:

:::: tabs

::: tab Server command

```yaml .rr.yaml
version: "3"

server:
  command: "php grpc-worker.php"

grpc:
  listen: "tcp://127.0.0.1:9001"

  proto:
    - "proto/pinger.proto"
```

> **Note**
> You can define command to start server in the `server.command` section:. It will be used to start PHP workers for all
> registered plugins, such as `grpc`, `http`, `jobs`, etc.

:::

::: tab Worker commands
You can also define command to start server in the `grpc.pool.command` section to separate server and grpc workers.

```yaml .rr.yaml
version: "3"

server:
  command: "php worker.php"

grpc:
  listen: "tcp://127.0.0.1:9001"

  pool:
    command: "php grpc-worker.php"

  proto:
    - "proto/pinger.proto"
```

:::
::::

> **Note**
> You can define multiple proto files in the `proto` section.


After configuring the server, you can start it using the following command:

```terminal
./rr serve
```

This will start the gRPC server and make the `Pinger` service available for remote clients to call. You can use any gRPC
client library in any language that supports gRPC to call the `ping` method.

## Metrics

RoadRunner has a [metrics plugin](../lab/metrics.md) that provides metrics for the gRPC server, which can be used with 
Prometheus and a preconfigured [Grafana dashboard](../lab/dashboards/grpc.md)

![grpc-metrics](https://user-images.githubusercontent.com/773481/235685443-05cf8af0-9e43-4aed-8801-da6595ca7d19.png)

## mTLS

To enable [mTLS](https://www.cloudflare.com/en-gb/learning/access-management/what-is-mutual-tls/) use the following
configuration:

```yaml
version: "3"

grpc:
  listen: "tcp://127.0.0.1:9001"

  proto:
    - "first.proto"
    - "second.proto"

  tls:
    key: "server-key.pem"
    cert: "server-cert.pem"
    root_ca: "rootCA.pem"
    client_auth_type: request_client_cert
```

Options for the `client_auth_type` are:

- `request_client_cert`
- `require_any_client_cert`
- `verify_client_cert_if_given`
- `require_and_verify_client_cert`
- `no_client_certs`

## Full example of Configuration

```yaml
version: "3"

grpc:
  # GRPC address to listen
  #
  # This option is required
  listen: "tcp://127.0.0.1:9001"

  # Proto file to use, multiply files supported [SINCE 2.6]. As of [2023.1.4], wilcards are allowed in the proto field.
  #
  # This option is required
  proto:
    - "*.proto"
    - "first.proto"
    - "second.proto"

  # GRPC TLS configuration
  #
  # This section is optional
  tls:
    # Path to the key file
    #
    # This option is required
    key: ""

    # Path to the certificate
    #
    # This option is required
    cert: ""

    # Path to the CA certificate, defines the set of root certificate authorities that servers use if required to verify a client certificate. Used with the `client_auth_type` option.
    #
    # This option is optional
    root_ca: ""

    # Client auth type.
    #
    # This option is optional. Default value: no_client_certs. Possible values: request_client_cert, require_any_client_cert, verify_client_cert_if_given, require_and_verify_client_cert, no_client_certs
    client_auth_type: no_client_certs

  # Maximum send message size
  #
  # This option is optional. Default value: 50 (MB)
  max_send_msg_size: 50

  # Maximum receive message size
  #
  # This option is optional. Default value: 50 (MB)
  max_recv_msg_size: 50

  # MaxConnectionIdle is a duration for the amount of time after which an
  #	idle connection would be closed by sending a GoAway. Idleness duration is
  #	defined since the most recent time the number of outstanding RPCs became
  #	zero or the connection establishment.
  #
  # This option is optional. Default value: infinity.
  max_connection_idle: 0s

  # MaxConnectionAge is a duration for the maximum amount of time a
  #	connection may exist before it will be closed by sending a GoAway. A
  #	random jitter of +/-10% will be added to MaxConnectionAge to spread out
  #	connection storms.
  #
  # This option is optional. Default value: infinity.
  max_connection_age: 0s

  # MaxConnectionAgeGrace is an additive period after MaxConnectionAge after
  #	which the connection will be forcibly closed.
  max_connection_age_grace: 0s8h

  # MaxConnectionAgeGrace is an additive period after MaxConnectionAge after
  #	which the connection will be forcibly closed.
  #
  # This option is optional: Default value: 10
  max_concurrent_streams: 10

  # After a duration of this time if the server doesn't see any activity it
  #	pings the client to see if the transport is still alive.
  #	If set below 1s, a minimum value of 1s will be used instead.
  #
  # This option is optional. Default value: 2h
  ping_time: 1s

  # After having pinged for keepalive check, the server waits for a duration
  #	of Timeout and if no activity is seen even after that the connection is
  #	closed.
  #
  # This option is optional. Default value: 20s
  timeout: 200s

  # Usual workers pool configuration
  pool:
    # Debug mode for the pool. In this mode, pool will not pre-allocate the worker. Worker (only 1, num_workers ignored) will be allocated right after the request arrived.
    #
    # Default: false
    debug: false

    # Override server's command
    #
    # Default: empty
    command: "php my-super-app.php"

    # How many worker processes will be started. Zero (or nothing) means the number of logical CPUs.
    #
    # Default: 0
    num_workers: 0

    # Maximal count of worker executions. Zero (or nothing) means no limit.
    #
    # Default: 0
    max_jobs: 0

    # Timeout for worker allocation. Zero means 60s.
    #
    # Default: 60s
    allocate_timeout: 60s

    # Timeout for the reset timeout. Zero means 60s.
    #
    # Default: 60s
    reset_timeout: 60s

    # Timeout for worker destroying before process killing. Zero means 60s.
    #
    # Default: 60s
    destroy_timeout: 60s
```

## Minimal dependencies

1. `Server` plugin for the worker pool.
2. `Logger` plugin to show log messages.
3. `Config` plugin to read and populate plugin's configuration.

## Common issues

1. Registering two services with the same name is not allowed. GRPC server will panic after that.
