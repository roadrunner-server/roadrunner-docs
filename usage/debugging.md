# Debugging
You can use RoadRunner scripts with xDebug extension. In order to enable configure your IDE to accept remote connections. 

Note, if you run multiple PHP processes you have to extend the maximum number of allowed connections to the number of active workers, otherwise some calls would not be caught on your breakpoints.

![xdebug](https://user-images.githubusercontent.com/796136/46493729-c767b400-c819-11e8-9110-505a256994b0.png)

> Do not forget to configure `xdebug.remote_enable=1` in php.ini and `xdebug.remote_autostart=1` if xDebug does not start automatically.

## Alternative methods
RoadRunner forward all the output from STDERR to user console when running with `-d` flag. This allows you to use standard `error_log` function to debug your application.

Since RoadRunner run as server application you can use functionality similar to Symfony VarDump Server out of the box. Any dump package which provides the ability to forward data to STDERR would work.

```
composer require spiral/dumper
```

```php
<?php
/**
 * @var Goridge\RelayInterface $relay
 */

use Spiral\Debug;
use Spiral\Goridge;
use Spiral\RoadRunner;

ini_set('display_errors', 'stderr');
require 'vendor/autoload.php';

$worker = new RoadRunner\Worker(new Goridge\StreamRelay(STDIN, STDOUT));
$psr7 = new RoadRunner\PSR7Client($worker);

$dumper = new Debug\Dumper();
$dumper->setRenderer(Debug\Dumper::ERROR_LOG, new Debug\Renderer\ConsoleRenderer());

while ($req = $psr7->acceptRequest()) {
    try {
        $resp = new \Zend\Diactoros\Response();
        $resp->getBody()->write("hello world");

        $dumper->dump($req->getUri(), Debug\Dumper::ERROR_LOG);

        $psr7->respond($resp);
    } catch (\Throwable $e) {
        $psr7->getWorker()->error((string)$e);
    }
}
```

![dump](https://user-images.githubusercontent.com/796136/46493853-0eee4000-c81a-11e8-9157-5bb5cfc5f97f.png)
