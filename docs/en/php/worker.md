# PHP Workers

In order to run your PHP application, you must create a worker endpoint and configure RoadRunner to use it. First,
install the required package using [Composer](https://getcomposer.org/).

```bash
composer require spiral/roadrunner nyholm/psr7
```

Simplest entrypoint with PSR-7 server API might looks like:

```php
<?php

use Spiral\RoadRunner;
use Nyholm\Psr7;

include "vendor/autoload.php";

$worker = RoadRunner\Worker::create();
$psrFactory = new Psr7\Factory\Psr17Factory();

$psr7 = new RoadRunner\Http\PSR7Worker($worker, $psrFactory, $psrFactory, $psrFactory);

while (true) {
    try {
        $request = $psr7->waitRequest();

        if (!($request instanceof \Psr\Http\Message\ServerRequestInterface)) { // Termination request received
            break;
        }
    } catch (\Throwable $e) {
        $psr7->respond(new Psr7\Response(400)); // Bad Request
        continue;
    }

    try {
        // Application code logic
        $psr7->respond(new Psr7\Response(200, [], 'Hello RoadRunner!'));
    } catch (\Throwable $e) {
        $psr7->respond(new Psr7\Response(500, [], 'Something Went Wrong!'));
    }
}
```

Such a worker will expect communication with the parent RoadRunner server over standard pipes, create `.rr.yaml` config
to enable it:

```yaml
server:
  command: "php psr-worker.php"

http:
  address: 0.0.0.0:8080
  pool:
    num_workers: 4
```

If you don't like `yaml` try `.rr.json`:

```json
{
  "server": {
    "command": "path-to-php/php psr-worker.php"
  },
  "http": {
    "address": "0.0.0.0:8080",
    "pool": {
      "num_workers": 4
    }
  }
}
```

You can start the application now by downloading the RR binary file and running `rr serve`

## Alternative Communication Methods

PHP Workers would utilize standard pipes STDOUT and STDERR to exchange data frames with RR server. In some cases you might
want to use alternative communication methods such as TCP socket:

```yaml
server:
  command: "php psr-worker.php"
  relay: "tcp://localhost:7000"
  
http:
  address: 0.0.0.0:8080
  pool:
    num_workers: 4
```

Unix sockets:

```yaml
server:
  command: "php psr-worker.php"
  relay:  "unix://rr.sock"

http:
  address: 0.0.0.0:8080
  pool:
    num_workers: 4
```

# Error Handling
There are multiple ways of how you can handle errors produces by PHP workers.

The simplest and most common way would be responding to parent service with the error message using `getWorker()->error()`:

```php
try {
    $resp = new Psr7\Response();
    $resp->getBody()->write("hello world");

    $psr7->respond($resp);
} catch (\Throwable $e) {
    $psr7->getWorker()->error((string)$e);
}
```

You can also flush your warning and errors into `STDERR` to output them directly into the console (similar to docker-compose).

```php
file_put_contents('php://stderr', 'my message');
```

Since RoadRunner 2.0 all warnings send to STDOUT will be forwarded to STDERR as well.

## Troubleshooting

In some cases, RR would not be able to handle errors produced by PHP worker (PHP is missing, the script is dead etc)
.

```
$ rr serve
```

You can troubleshoot it by invoking `command` set in `.rr` file manually:

```
$ php psr-worker.php
```

The worker should not cause any error until any input provided and must fail with invalid input signature after the
first input character.

## Other Type of Workers

Different roadrunner implementations might define their own worker APIs,
examples: [GRPC](https://github.com/spiral/roadrunner-grpc), [Workflow/Activity Worker](https://docs.temporal.io/docs/php/workers/).
