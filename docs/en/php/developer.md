# PHP Workers — Developer Mode

When RoadRunner starts workers, they operate in daemon mode. In this mode, you need to reload the server every time you
make changes to your codebase.

## Manual restarting

One of the way reloading server is using console command:

```terminal
./rr reset
```

This command is also helpful when you need to restart a remote RoadRunner server using a local RoadRunner
client.

### Restarting in Docker container

For example, you can use this command to restart workers inside a Docker container. To do this, you need to
configure the RoadRunner server to handle external RPC requests by adding the following lines to your configuration
file:

```yaml .rr.yaml
rpc:
  listen: tcp://:6001
```

> **Note**
> You must also forward/expose port `6001` in your Docker container to be able to use this feature.

Now when you run the command, RoadRunner client sends RPC request to the running server.

> **Warning**
> Pay attention to the RPC host and port which uses RoadRunner client specified in the `.rr.yaml` should be the same as
> the RPC host and port which uses RoadRunner server. By default, client uses `127.0.0.1:6001`.

## Debug Mode

It can be a time-consuming and tedious process to restart workers manually, especially during the development phase
of a project. To address this issue, RoadRunner provides a debug mode that automatically restarts workers after each
handled request, allowing developers to make changes to their codebase without having to manually reload the server each
time.

To enable debug mode, you can set the `pool.debug` option to `true` in desired plugin section that has workers pool:

```yaml
http:
  pool:
    debug: true
    num_workers: 4
```

Or if you have only `debug` option in the `pool` section you can use short syntax:

```yaml
http:
  pool.debug: true
```

> **Note**
> Every plugin in RoadRunner that creates workers has a `pool` section in which you can activate debug mode.


> **Warning**
> When using the `pool.debug` option in RoadRunner, it is important to note that settings in `pool` section would work
> differently. All options will be ignored (`supervisor`, `max_jobs`, `num_workers`, etc). This is because, in debug 
> mode, RoadRunner does not create a worker at startup. Instead, it waits for requests to come in and creates workers 
> accordingly. After the response, RoadRunner stops and removes the worker.
> When you send 2-3-n parallel requests to RoadRunner, it creates 2-3-n workers to handle those requests simultaneously.
> The number of workers depends on the number of requests you send. Similarly, when you use the Jobs plugin and the Jobs
> consumers, every message consumed creates a worker to handle that message. The number of workers is based on the
> number of messages consumed.
>
> This enables you to make changes to your codebase and reload it automatically.

## Stop Command

In RoadRunner, you can send a `stop` command from the worker to the parent server to force process destruction. When
this happens, the job/request will be automatically forwarded to the next worker in the queue.

You can use this feature to implement `max_jobs` control on the PHP side. This can be useful for controlling memory
usage inside the PHP script or for managing long-running tasks that need to be periodically restarted.

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

As you can see in the example above, we send a `stop` command after handling 10 requests, to force process destruction.
This ensures that the script does not consume too much memory and avoids any potential memory leaks.
