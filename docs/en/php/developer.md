# Developer Mode

RoadRunner uses PHP scripts in daemon mode. This means that you have to reload the server every time you change your
codebase, or configure Roadrunner to do that automatically.

## In Docker

You can reset rr process in docker by connecting to it using local rr client.

```yaml
rpc:
  listen: tcp://:6001
```

> Make sure to forward/expose port 6001.

Then run `rr reset` locally on file change.

## Debug Mode

To run workers in debug mode (similar to how PHP-FPM operates):

```yaml
http:
  pool.debug: true
```

# Restarting Workers

RoadRunner provides multiple ways to safely restart worker(s) on demand. Both approaches can be used on a live server
and should not cause downtime.

## Stop Command

You are able to send a `stop` command from the worker to the parent server to force process destruction. In this
scenario,
the job/request will be automatically forwarded to the next worker.

We can demonstrate it by implementing `max_jobs` control on PHP end:

```php
<?php

use Spiral\RoadRunner;
use Nyholm\Psr7;

include "vendor/autoload.php";

$worker = RoadRunner\Worker::create();
$psrFactory = new Psr7\Factory\Psr17Factory();

$worker = new RoadRunner\Http\PSR7Worker($worker, $psrFactory, $psrFactory, $psrFactory);

$count = 0;
while ($req = $worker->waitRequest()) {
    try {
        $rsp = new Psr7\Response();
        $rsp->getBody()->write('Hello world!');

        $count++;
        if ($count > 10) {
            $worker->getWorker()->stop();
            return;
        }

        $worker->respond($rsp);
    } catch (\Throwable $e) {
        $worker->getWorker()->error((string)$e);
    }
}
```

> **INFO**
> This approach can be used to control memory usage inside the PHP script.