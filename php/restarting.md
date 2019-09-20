# Restarting Workers
RoadRunner provides multiple ways to safely restart worker(s) on demand. Both approaches can be used on a live server and should not cause downtime.

## Stop Command
You are able to send `stop` command from worker to parent server to force process destruction. In this scenario, the job/request will be automatically forwarded to the next worker.

We can demonstrate it by implementing `maxJobs` control on PHP end:

```php
<?php
ini_set('display_errors', 'stderr');
include "vendor/autoload.php";

$relay = new Spiral\Goridge\StreamRelay(STDIN, STDOUT);
$psr7 = new Spiral\RoadRunner\PSR7Client(new Spiral\RoadRunner\Worker($relay));

$count = 0;
while ($req = $psr7->acceptRequest()) {
    if ($count++ > 1000) {
        $psr7->getWorker()->stop();
        return;
    }

    try {
        $resp = new \Zend\Diactoros\Response();
        $resp->getBody()->write("hello world");

        $psr7->respond($resp);
    } catch (\Throwable $e) {
        $psr7->getWorker()->error((string)$e);
    }
}
```

> This approach can be used to control memory usage inside the PHP script.

## Full Reset
You can also initiate a rebuild of all RoadRunner workers using embedded [RPC bus](RPC-Integration):

```php
$rpc = new Goridge\RPC(new Spiral\Goridge\SocketRelay("127.0.0.1", 6001));
$rpc->call("http.Reset", true);
```
