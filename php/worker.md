# PHP Workers
In order to run your PHP application, you must create a worker endpoint and configure RoadRunner to use it. First, install the required package using [Composer](https://getcomposer.org/).

```bash
composer require spiral/roadrunner
```

Simpliest enterpoint with PSR-7 server API might looks like:

```php
<?php
/**
 * @var Goridge\RelayInterface $relay
 */
use Spiral\Goridge;
use Spiral\RoadRunner;

ini_set('display_errors', 'stderr');
require 'vendor/autoload.php';

$worker = new RoadRunner\Worker(new Goridge\StreamRelay(STDIN, STDOUT));
$psr7 = new RoadRunner\PSR7Client($worker);

while ($req = $psr7->acceptRequest()) {
    try {
        $resp = new \Zend\Diactoros\Response();
        $resp->getBody()->write("hello world");

        $psr7->respond($resp);
    } catch (\Throwable $e) {
        $psr7->getWorker()->error((string)$e);
    }
}
```

Such worker will expect communication with parent RoadRunner server over standard pipes, create`.rr.yaml` config to enable it:

```yaml
http:
  address:    0.0.0.0:8080
  workers:
    command:  "php psr-worker.php"
    pool:
      numWorkers: 4
```

If you don't like `yaml` try `.rr.json`:

```json
{
  "http": {
    "address": "0.0.0.0:8080",
    "workers": {
      "command": "path-to-php/php psr-worker.php",
      "pool": {
        "numWorkers": 4
      }
    }
  }
}
```

You can start the application now by downloading the RR binary file and running `rr serve -v -d`

## Alternative Communication Methods
PHP Workers would utilize standard pipes STDOUT and STDERR to exchange data frames with RR server, echoing any data to this channels will break the communication (i.e. no unbuffered `echo` or `headers`).

If your application must write to STDOUT use alternative communication method over TCP:

```yaml
http:
  address:    0.0.0.0:8080
  workers:
    command:  "php psr-worker.php"
    relay:    "tcp://localhost:7000"
    pool:
      numWorkers: 4
```

```php
$worker = new RoadRunner\Worker(
    new Goridge\SocketRelay("localhost", 7000)
);
```

Unix sockets:

```yaml
http:
  address:    0.0.0.0:8080
  workers:
    command:  "php psr-worker.php"
    relay:    "unix://rr.sock"
    pool:
      numWorkers: 4
```

```php
$worker = new RoadRunner\Worker(
    new Goridge\SocketRelay("rr.sock", null, Goridge\SocketRelay::SOCK_UNIX)
);
```

## Troubleshooting
In some cases, RR would not be able to handle errors produced by PHP worker (PHP is missing, the script is dead and etc).

```
rr serve -v -d
DEBU[0003] [rpc]: started
DEBU[0003] [http]: started
ERRO[0003] [http]: unable to connect to worker: unexpected response, header is missing: exit status 1
DEBU[0003] [rpc]: stopped
```

You can troubleshoot it by invoking `command` set in `.rr` file manually:

```
$ php psr-worker.php
```

The worker should not cause any error until any input provided and must fail with invalid input signature after the first input character.

## Other Type of Workers
Different roadrunner implementations might define their own worker APIs, examples: [AWS Lambda](https://github.com/spiral/roadrunner/wiki/AWS-Lambda), [GRPC](https://github.com/spiral/php-grpc).
