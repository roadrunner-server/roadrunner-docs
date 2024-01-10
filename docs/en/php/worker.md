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
You should choose the appropriate plugin based on the requirements of your application. In the examples below, 
we will explore the creation of an HTTP worker and a simple implementation of an entry point that can handle several
types of requests. 

### Simple HTTP Worker

To create HTTP worker, you need to install the required composer packages:

```terminal
composer require spiral/roadrunner-http nyholm/psr7
```

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
        if ($request === null) {
            break;
        }
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

### Single entry point

In the example below, we will create a single entry point capable of processing incoming HTTP requests and requests
from the queue system. RoadRunner provides the `RR_MODE` environment variable, which allows us to determine the type
of request received. We can then instantiate the appropriate worker, process the incoming request, return the response, 
and handle any potential exceptions. In this example, we strive to minimize dependencies and reduce the amount of code
to the maximum extent possible. The sole objective of this example is to demonstrate how to handle various types of 
requests.

First of all, we need to install the required composer packages:

```terminal
composer require spiral/roadrunner-http spiral/roadrunner-jobs nyholm/psr7
```

Let's start by creating an enum to enumerate the possible operating modes of RoadRunner. In this example, we will only 
require the values **http** and **jobs**, but we will list all available modes:

```php RoadRunnerMode.php
namespace App;

enum RoadRunnerMode: string
{
    case Http = 'http';
    case Jobs = 'jobs';
    case Temporal = 'temporal';
    case Grpc = 'grpc';
    case Tcp = 'tcp';
    case Centrifuge = 'centrifuge';
    case Unknown = 'unknown';
}
```

To split the logic for handling different types of requests, let's introduce the concept of a **dispatcher**. 
It will determine whether it can handle an incoming request and process it. We'll define an interface for dispatchers,
which will include two methods: **canServe** - responsible for determining if the dispatcher can handle the request or not,
and **serve** - intended to process the request if the **canServe** method returns **true**:

```php DispatcherInterface.php
namespace App\Dispatcher;

use Spiral\RoadRunner\EnvironmentInterface;

interface DispatcherInterface
{
    public function canServe(EnvironmentInterface $env): bool;

    public function serve(): void;
}
```

Let's implement the two dispatchers we need. One for handling HTTP requests and the other for processing requests
from the queuing system:

```php HttpDispatcher.php
namespace App\Dispatcher;

use App\RoadRunnerMode;
use Nyholm\Psr7\Factory\Psr17Factory;
use Nyholm\Psr7\Response;
use Spiral\RoadRunner\EnvironmentInterface;
use Spiral\RoadRunner\Http\PSR7Worker;
use Spiral\RoadRunner\Worker;

final class HttpDispatcher implements DispatcherInterface
{
    public function canServe(EnvironmentInterface $env): bool
    {
        return $env->getMode() === RoadRunnerMode::Http->value;
    }

    public function serve(): void
    {
        $factory = new Psr17Factory();
        $worker = new PSR7Worker(Worker::create(), $factory, $factory, $factory);

        while (true) {
            try {
                $request = $worker->waitRequest();
                if ($request === null) {
                    break;
                }
            } catch (\Throwable $e) {
                $worker->respond(new Response(400));
                continue;
            }

            try {
                // Handle request and return response.
                $worker->respond(new Response(200, [], 'Hello RoadRunner!'));
            } catch (\Throwable $e) {
                $worker->respond(new Response(500, [], 'Something Went Wrong!'));
                $worker->getWorker()->error((string)$e);
            }
        }
    }
}
```

```php QueueDispatcher.php
namespace App\Dispatcher;

use App\RoadRunnerMode;
use Spiral\RoadRunner\EnvironmentInterface;
use Spiral\RoadRunner\Jobs\Consumer;

final class QueueDispatcher implements DispatcherInterface
{
    public function canServe(EnvironmentInterface $env): bool
    {
        return $env->getMode() === RoadRunnerMode::Jobs->value;
    }

    public function serve(): void
    {
        $consumer = new Consumer();

        while ($task = $consumer->waitTask()) {
            try {
                // Handle and process task. Here we just print payload.
                var_dump($task);

                // Complete task.
                $task->complete();
            } catch (\Throwable $e) {
                $task->fail($e);
            }
        }
    }
}
```

In the `canServe` method of both dispatchers, we use the `Spiral\RoadRunner\EnvironmentInterface` interface provided by
the [spiral/roadrunner-worker](https://github.com/roadrunner-php/worker) package. This interface provides the `getMode`
method, which returns the current mode of RoadRunner as a string. The implementation of this interface, provided by the package,
determines the mode based on the `RR_MODE` environment variable.

The `serve` method creates the worker and processes the incoming request. In the **HttpDispatcher**, we've used code
from the above HTTP worker example, while in the **QueueDispatcher**, we instantiate the `Spiral\RoadRunner\Jobs\Consumer`
class provided by the [spiral/roadrunner-jobs](https://github.com/roadrunner-php/jobs) package. In a loop, we retrieve
and handle incoming tasks. Both methods are significantly simplified, lacking real processing logic for requests.
They send a string response to the browser and output the task object to the console.

Now, let's create an entry point that will be specified in the RoadRunner configuration file and will use our dispatchers:

```php app.php
require __DIR__ . '/vendor/autoload.php';

use App\Dispatcher\DispatcherInterface;
use App\Dispatcher\HttpDispatcher;
use App\Dispatcher\QueueDispatcher;
use Spiral\RoadRunner\Environment;

/**
 * Collect all dispatchers.
 *
 * @var DispatcherInterface[] $dispatchers
 */
$dispatchers = [
    new HttpDispatcher(),
    new QueueDispatcher(),
];

// Create environment
$env = Environment::fromGlobals();

// Execute dispatcher that can serve the request
foreach ($dispatchers as $dispatcher) {
    if ($dispatcher->canServe($env)) {
        $dispatcher->serve();
    }
}
```

> **Note**
> The [Roadrunner Bridge](https://github.com/spiral/roadrunner-bridge) package, which provides the integration of **RoadRunner**
> into the **Spiral Framework** embodies this concept, and you may explore it if you wish to delve deeper into this topic.

Create a `.rr.yaml` configuration file:

```yaml .rr.yaml
version: '3'

rpc:
  listen: tcp://127.0.0.1:6001

server:
  command: "php app.php"

http:
  address: "0.0.0.0:8080"

jobs:
  consume: [ "default" ]
  pool:
    num_workers: 2
    supervisor:
      max_worker_memory: 100
  pipelines:
    default:
      driver: memory
      config:
        priority: 10
```

> **Note**
> Read more about the [RoadRunner configuration](../intro/config.md).

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
