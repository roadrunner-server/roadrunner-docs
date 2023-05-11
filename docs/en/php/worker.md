# PHP Workers

RoadRunner is a high-performance PHP application server designed to handle a large number of requests simultaneously. It
does so by running your PHP application in the form of workers, which follow the shared-nothing architecture. Each
worker represents an individual process, ensuring isolation and independence in their operation.

In this section, you will learn how to create a PHP worker that handles HTTP requests and returns a response to the
RoadRunner server.

## Creating a worker

### Worker types

RoadRunner provides several plugins that use workers to receive requests,
including [HTTP](https://github.com/roadrunner-php/http), [Jobs](https://github.com/roadrunner-php/jobs),
[Centrifuge](https://github.com/roadrunner-php/centrifugo), [gRPC](https://github.com/roadrunner-php/grpc),
[TCP](https://github.com/roadrunner-php/tcp), and [Temporal](https://legacy-documentation-sdks.temporal.io/php/workers).
You should choose the appropriate plugin based on the requirements of your application. In our example, we will create
a worker for the HTTP plugin.

To create HTTP worker, you need to install the required composer packages:

```terminal
composer require spiral/roadrunner-http nyholm/psr7
```

### Entry point

After installing the required packages, you can create a worker. Here is an example of the simplest entry point with the
PSR-7 server API:

```php psr-worker.php
<?php

require __DIR__ . '/vendor/autoload.php';

use Nyholm\Psr7\Response;
use Nyholm\Psr7\Factory\Psr17Factory;

use Spiral\RoadRunner\Worker;
use Spiral\RoadRunner\Http\PSR7Worker;


// Create new RoadRunner worker from global environment
$worker = Worker::create();

// Create common PSR-17 HTTP factory
$factory = new Psr17Factory();

$psr7 = new PSR7Worker($worker, $factory, $factory, $factory);

while (true) {
    try {
        $request = $psr7->waitRequest();
    } catch (\Throwable $e) {
        // Although the PSR-17 specification clearly states that there can be
        // no exceptions when creating a request, however, some implementations
        // may violate this rule. Therefore, it is recommended to process the 
        // incoming request for errors.
        //
        // Send "Bad Request" response.
        $psr7->respond(new Response(400));
        continue;
    }

    try {
        // Here is where the call to your application code will be located. 
        // For example:
        //  $response = $app->send($request);
        //
        // Reply by the 200 OK response
        $psr7->respond(new Response(200, [], 'Hello RoadRunner!'));
    } catch (\Throwable $e) {
        // In case of any exceptions in the application code, you should handle
        // them and inform the client about the presence of a server error.
        //
        // Reply by the 500 Internal Server Error response
        $psr7->respond(new Response(500, [], 'Something Went Wrong!'));
        
        // Additionally, we can inform the RoadRunner that the processing 
        // of the request failed.
        $psr7->getWorker()->error((string)$e);
    }
}
```

This worker expects communication with the RoadRunner server over standard pipes.

Create a `.rr.yaml` configuration file to enable it:

```yaml .rr.yaml
server:
  command: "php psr-worker.php"

http:
  address: 0.0.0.0:8080
```

> **Note**
> Read more about the configuration HTTP in the [HTTP Plugin](../http/http.md) section.

Now you can start the RoadRunner server by running the following command:

```terminal
./rr serve
```

> **Note**
> Read more about how to download and install RoadRunner in the [RoadRunner — Installation](../intro/install.md)
> section.

### Error Handling

There are multiple ways to handle errors produced by PHP workers in RoadRunner. The simplest and most common way is to
respond to the parent service with the error message using `$psr7->getWorker()->error()` method.

**Here's an example:**

```php psr-worker.php
try {
    $resp = new Psr7\Response();
    $resp->getBody()->write("hello world");

    $psr7->respond($resp);
} catch (\Throwable $e) {
    $psr7->getWorker()->error((string)$e);
}
```

Another way to handle errors is to flush warnings and errors to `STDERR` to output them directly into the console.

**Here's an example:**

```php
file_put_contents('php://stderr', 'Error message');
```

> **Note**
> Since RoadRunner 2.0 all warnings send to `STDOUT` will be forwarded to `STDERR` as well.

## Communication Methods

By default, workers use standard pipes `STDOUT` and `STDERR` to exchange data frames with the RoadRunner server.
However, in some cases, you may want to use alternative communication methods such as `TCP` or `unix` sockets.

:::: tabs

::: tab TCP Sockets

```yaml .rr.yaml
server:
  command: "php psr-worker.php"
  relay: "tcp://127.0.0.1:7000"

http:
  address: 0.0.0.0:8080
```

:::

::: tab Unix Sockets

```yaml .rr.yaml
server:
  command: "php psr-worker.php"
  relay: "unix://rr.sock"

http:
  address: 0.0.0.0:8080
```

:::

::::

## Process supervision

RoadRunner provides process supervision capabilities to monitor your application and perform a soft reset between
requests if necessary. You can configure process supervision for your application by editing your `.rr.yaml`
configuration file and specifying the necessary limits.

**Here's an example configuration**

```yaml .rr.yaml
http:
  address: "0.0.0.0:8080"
  pool:
    supervisor:

      # watch_tick defines how often to check the state of the workers (seconds)
      watch_tick: 1s

      # ttl defines maximum time worker is allowed to live (seconds)
      ttl: 0

      # idle_ttl defines maximum duration worker can spend in idle mode after 
      # first use. Disabled when 0 (seconds)
      idle_ttl: 10s

      # exec_ttl defines maximum lifetime per job (seconds)
      exec_ttl: 10s

      # max_worker_memory limits memory usage per worker (MB)
      max_worker_memory: 100

    num_workers: 6
```

> **Warning**
> Please pay attention, that the previous section name was **limit**, current - **supervisor**

## Troubleshooting

In some cases, RoadRunner may not be able to handle errors produced by the PHP worker, such as if PHP is missing or the
script has died.

```terminal
./rr serve
```

If this happens, you can troubleshoot the issue by invoking the `command` specified in your `.rr.yaml` file manually.

```terminal
php psr-worker.php
```

If there are any errors or issues with the script, they should be visible in the output of this command.

The worker should not cause any errors until any input is provided, and it should fail with an invalid input signature
after the first input character. If this is not the case, there may be an issue with the PHP script or the way it is
interacting with RoadRunner.

## What's Next?

1. [Plugins — Server](../plugins/server.md) - Read more about RoadRunner server plugin.