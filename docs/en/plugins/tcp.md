# Plugins — TCP

The RoadRunner TCP plugin helps you handle TCP requests. You can use this plugin to make your own servers like an SMTP
server, and send TCP requests directly to PHP workers for handling.

## Principle of work

The RoadRunner TCP plugin operates by receiving client requests and proxying them to free PHP workers that are not
currently processing any request.

> **Warning**
> It's essential to note that PHP workers are stateless, meaning you cannot save the context of a request between
> multiple requests from a single client. Instead, you must use external storage, for
> example [Key Value](../kv/overview.md), to maintain context and rely on the connection UUID to identify requests from
> individual clients.

The request sent to the PHP worker contains the following context:

- `event` - The event type. The following events are supported:
    - `CONNECTED`: Sent when a client connects to the server.
    - `DATA`: Sent when a client sends data to the server.
    - `CLOSED`: Sent when a client disconnects from the server.
- `remote_addr`: The client's IP address.
- `server`: The server name, as specified in the RoadRunner configuration (e.g., `smtp`, `server2`).
- `body`: The request body.
- `uuid`: The connection UUID.

The protocol used by the RoadRunner TCP plugin provides a bidirectional communication channel between the PHP worker
and the RoadRunner server. This design allows PHP workers to send responses back to the client, enabling seamless
handling of client requests and communication between all parties.

## Configuration

The TCP plugin is configured via the `tcp` section of the RoadRunner configuration file.

**Here is an example configuration:**

:::: tabs

::: tab Server command

```yaml .rr.yaml
server:
  command: "php tcp-worker.php"

tcp:
  servers:
    smtp:
      addr: tcp://127.0.0.1:1025
      delimiter: "\r\n" # by default

    server2:
      addr: tcp://127.0.0.1:8889

  pool:
    num_workers: 2
    max_jobs: 0
    allocate_timeout: 60s
    destroy_timeout: 60s
```

> **Note**
> You can define command to start server in the `server.command` section:. It will be used to start PHP workers for all
> registered plugins, such as `grpc`, `http`, `jobs`, etc.

:::

::: tab Worker commands
You can also define command to start server in the `grpc.pool.command` section to separate server and grpc workers.

```yaml .rr.yaml
server:
  command: "php worker.php"

tcp:
  servers:
    smtp:
      addr: tcp://127.0.0.1:1025
      delimiter: "\r\n" # by default

    server2:
      addr: tcp://127.0.0.1:8889

  pool:
    command: "php tcp-worker.php"
    num_workers: 2
    max_jobs: 0
    allocate_timeout: 60s
    destroy_timeout: 60s
```

:::
::::

#### Configuration Parameters

- `servers`: A list of TCP servers to start. Each server should contain the following keys:
    - `addr`: The server address and port, specified in the format `tcp://<IP_ADDRESS>:<PORT>`.
    - `delimiter`: (Optional) The data packet delimiter. By default, it is set to `\r\n`. Each data packet should end
      either with an `EOF` or the specified delimiter.
    - `read_buf_size`: (Optional) The size of the read buffer in MB. To reduce the number of read syscalls, consider
      using a larger buffer size if you expect to receive large payloads on the TCP server.

- `pool`: Configuration for the PHP worker pool.
    - `num_workers`: The number of PHP workers to allocate.
    - `max_jobs`: The maximum number of jobs each worker can handle. Set to 0 for no limit.
    - `allocate_timeout`: The timeout for worker allocation, specified in the format `<VALUE>s` (e.g., `60s` for 60
      seconds).
    - `destroy_timeout`: The timeout for worker destruction, specified in the same format as allocate_timeout.

## PHP client

The RoadRunner TCP plugin comes with a convenient PHP package that simplifies the process of integrating the plugin with
your PHP application.

### Installation

To get started, you can install the package via Composer using the following command:

```terminal
composer require spiral/roadrunner-tcp
```

### Usage

The following example demonstrates how to create a simple PHP worker:

```php tcp-worker.php
require __DIR__ . '/vendor/autoload.php';

use Spiral\RoadRunner\Worker;
use Spiral\RoadRunner\Tcp\TcpWorker;
use Spiral\RoadRunner\Tcp\TcpResponse;
use Spiral\RoadRunner\Tcp\TcpEvent;

// Create new RoadRunner worker from global environment
$worker = Worker::create();

$tcpWorker = new TcpWorker($worker);

while ($request = $tcpWorker->waitRequest()) {

    try {
        if ($request->event === TcpEvent::Connected) {
            // You can close connection according to your restrictions
            if ($request->remoteAddr !== '127.0.0.1') {
                $tcpWorker->close();
                continue;
            }

            // Or continue reading data from the server
            // By default, the server closes the connection if a worker
            // doesn't send a CONTINUE response
            $tcpWorker->read();

            // Or send a response to the TCP connection, for example, to an SMTP client
            $tcpWorker->respond("220 mailamie \r\n");

        } elseif ($request->event === TcpEvent::Data) {

            $body = $request->body;

            // Handle request from TCP server [tcp_access_point_1]
            if ($request->server === 'tcp_access_point_1') {

                // Send response and close connection
                $tcpWorker->respond('Access denied', TcpResponse::RespondClose);

            // Handle request from TCP server [server2]
            } elseif ($request->server === 'server2') {

                // Send response to the TCP connection and wait for the next request
                $tcpWorker->respond(\json_encode([
                    'remote_addr' => $request->remoteAddr,
                    'server' => $request->server,
                    'uuid' => $request->connectionUuid,
                    'body' => $request->body,
                    'event' => $request->event
                ]));
            }

        // Handle closed connection event
        } elseif ($request->event === TcpEvent::Close) {
            // Do something

            // You don't need to send a response on a closed connection
        }

    } catch (\Throwable $e) {
        $tcpWorker->respond("Something went wrong\r\n", TcpResponse::RespondClose);
        $worker->error((string)$e);
    }
}
```

## What's Next?

1. [Plugins — KV](../kv/overview.md) - Learn how to use the Key Value plugin to store data between requests.
