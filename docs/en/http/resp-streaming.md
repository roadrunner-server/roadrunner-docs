# HTTP Response streaming `[>=2023.3]`

RoadRunner supports HTTP response streaming. This feature means that responses can be sent to the client in chunks. It is useful when you need to send a large amount of data to the client.
You don't need to update the configuration to enable this feature. It is enabled by default and controlled by the PHP worker.

## Samples

### Sending a response in chunks

The size of the chunks is controlled by the PHP worker. You can send a chunk by calling the `respond()` method of the `Spiral\RoadRunner\Http\HttpWorker` class. 
The signature of the method is the following:
```php
    /**
     * @throws \JsonException
     */
    public function respond(int $status, string|Generator $body = '', array $headers = [], bool $endOfStream = true): void
```

The `$body` parameter can be a string or a generator. If it is a generator, the worker will iterate over it and send chunks to the client.
`$status` and `$headers` are the same as in the `respond()` method of the `Spiral\RoadRunner\Http\HttpWorker` class.
The `$endOfStream` parameter indicates whether the response is finished. If set to false, the worker will wait for the next chunk.

Here is the example of the streaming response:

```php
<?php

require __DIR__ . '/vendor/autoload.php';

use Spiral\RoadRunner;

ini_set('display_errors', 'stderr');
require __DIR__ . "/vendor/autoload.php";

$worker = RoadRunner\Worker::create();
$http = new RoadRunner\Http\HttpWorker($worker);
$read = static function (): Generator {
    foreach (\file(__DIR__ . '/test.txt') as $line) {
        try {
            yield $line;
        } catch (Spiral\RoadRunner\Http\Exception\StreamStoppedException) {
            // Just stop sending data
            return;
        }
    }
};

try {
    while ($req = $http->waitRequest()) {
        $http->respond(200, $read());
    }
} catch (\Throwable $e) {
    $worker->error($e->getMessage());
}
```

### Sending headers and status codes

You can send headers and status codes (`1XX` multiple times, or other, but only once) to the client during the streaming.

```php
<?php

use Spiral\RoadRunner;

ini_set('display_errors', 'stderr');
require __DIR__ . "/vendor/autoload.php";

$worker = RoadRunner\Worker::create();
$http = new RoadRunner\Http\HttpWorker($worker);
$read = static function (): Generator {
    $limit = 10;
    foreach (\file(__DIR__ . '/test.txt') as $line) {
        foreach (explode('"', $line) as $chunk) {
            try {
                usleep(50_000);
                yield $chunk;
            } catch (Spiral\RoadRunner\Http\Exception\StreamStoppedException $e) {
                // Just stop sending data
                return;
            }
            if (--$limit === 0) {
                return;
            }
        }
    }
};


try {
    while ($req = $http->waitRequest()) {
        $http->respond(100, '', headers: ['X-100' => ['100']], endOfStream: false);
        $http->respond(101, '', headers: ['X-101' => ['101']], endOfStream: false);
        $http->respond(102, '', headers: ['X-102' => ['102']], endOfStream: false);
        $http->respond(103, '', headers: ['Link' => ['</style111.css>; rel=preload; as=style'], 'X-103' => ['103']], endOfStream: false);
        $http->respond(200, $read(), headers: ['X-200' => ['200']], endOfStream: true);
    }
} catch (\Throwable $e) {
    $worker->error($e->getMessage());
}
```

In this example, we send 5 status codes and 5 headers to the client. You may send a `103 Early Hints` status code (or any `1XX` status code) to the client at any time during streaming (do not forget about `$endOfStream`).