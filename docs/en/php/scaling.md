# Dynamic Worker Scaling

## Introduction

This feature became available starting from the RoadRunner `2023.3` release.
Users can now scale their RoadRunner workers dynamically via RPC.
A new class, `Spiral\RoadRunner\WorkerPool`, has been introduced to provide an easy interface to **add** or **remove** 
workers from the RoadRunner workers pool.

### Limitations
- This feature is not available when running RoadRunner in debug mode (`pool.debug=true`).

### Usage

Below is a brief example demonstrating how to use this new feature:

```php
use Spiral\RoadRunner\WorkerPool;
use Spiral\Goridge\RPC\RPC;

$rpc = RPC::create('tcp://127.0.0.1:6001');
$pool = new WorkerPool($rpc);

// Add a worker to the pool.
$pool->addWorker('http');

// Remove a worker from the pool.
$pool->removeWorker('http');
```

### List of the supported plugins:
- `http`, `grpc`, `temporal`, `centrifuge`, `tcp`, `jobs`.

This provides developers more control and flexibility over their RoadRunner setup, 
allowing for better resource allocation based on the needs of their application.
