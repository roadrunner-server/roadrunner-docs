# Plugins — Server

RoadRunner server plugin, is responsible for starting worker pools for plugins that use workers, such as `http`, `tcp`,
`jobs`, `centrifuge`, `temporal`, and `grpc`. The worker pools inherit all of RoadRunner's features, such as
supervising, state machine, and command handling.

## Configuration

The `server` section contains various options for configuring the plugin.

**Here is an example of a configuration:**

```yaml .rr.yaml
server:
  on_init:
    # Command to execute before the main server's command
    #
    # This option is required if using on_init
    command: "any php or script here"

    # Username (not UID) of the user from whom the on_init command is executed. An empty value means to use the RR process user.
    #
    # Default: ""
    user: ""

    # Script execute timeout
    #
    # Default: 60s [60m, 60h], if used w/o units its means - NANOSECONDS.
    exec_timeout: 20s

    # Environment variables for the worker processes.
    #
    # Default: <empty map>
    env:
      - SOME_KEY: "SOME_VALUE"
      - SOME_KEY2: "SOME_VALUE2"

  # Worker starting command, with any required arguments.
  #
  # This option is required.
  command: "php psr-worker.php"

  # Username (not UID) for the worker processes. An empty value means to use the RR process user.
  #
  # Default: ""
  user: ""

  # Group name (not GID) for the worker processes. An empty value means to use the RR process user.
  #
  # Default: ""
  group: ""

  # Environment variables for the worker processes.
  #
  # Default: <empty map>
  env:
    - SOME_KEY: "SOME_VALUE"
    - SOME_KEY2: "SOME_VALUE2"

  relay: pipes

  # Timeout for relay connection establishing (only for socket and TCP port relay).
  #
  # Default: 60s
  relay_timeout: 60s
```

> **Note**
> Worker relay can be: `pipes`, TCP (eg.: `tcp://127.0.0.1:6002`), or socket (eg.: `unix:///var/run/rr.sock`). But in
> most cases, you should use the default `pipes` relay, it is the fastest communication transport.
> It uses an inter-process communication mechanism that allows for fast and efficient communication between
> processes. It does not require any network connections or external libraries, making it a lightweight and fast option.

> **Note**
> Use `on_init.user` option to execute the on_init command under a different user.

### Server initialization

The `on_init` section is used for application initialization or warming up before starting workers. It allows you to set
a command script that will be executed before starting the workers. You can also set environment variables to pass to
this script.

> **Note**
> If the `on_init` command fails (i.e., returns a non-zero exit code), RoadRunner will log the error but continue
> execution. This ensures that a failure during initialization does not interrupt the application's operation.

### Worker starting command

The `server.command` option is required and is used to start the worker pool for each configured section in the config.

> **Note**
> This option can be overridden by plugins with a pool section, such as the `http.pool.command`, or in general `<plugin>.pool.command`.

The `user` and `group` options allow you to set the user and group that will start and own the worker process. This
feature provides an additional layer of security and control over the application's execution environment.

> **Note**
> An empty value means to use the RoadRunner process user.

> **Warning**
> RoadRunner must be started from the root user. Root access is needed only to fork the process under a
> different user. Once the worker process is started, it will run with the specified user and group permissions,
> providing a secure and controlled execution environment for the application. All temporary files (`http` for example)
> would be created with the provided user/group

The `env` option allows you to set environment variables to pass to the worker script.

## PHP Client

There is a package that simplifies the process of integrating the RoadRunner worker pool with a PHP application. The
package contains a common codebase for all RoadRunner workers, making it easy to integrate and communicate with workers
created by RoadRunner.

When RoadRunner creates workers for any plugin that uses workers, it runs a PHP script and starts communicating with it
using a relay. The relay can be `pipes`, `TCP`, or a `socket`. The PHP client simplifies this process by providing a
convenient interface for sending and receiving payloads to and from the worker.

**Here is an example of simple PHP worker:**

```php worker.php
<?php

require __DIR__ . '/vendor/autoload.php';

// Create a new Worker from global environment
$worker = \Spiral\RoadRunner\Worker::create();

while ($data = $worker->waitPayload()) {
    // Received Payload
    var_dump($data);

    // Respond Answer
    $worker->respond(new \Spiral\RoadRunner\Payload('DONE'));
}
```

The worker waits for incoming payloads, for example `HTTP request`, `TCP request`, `Queue task` or`Centrifuge message`,
processes them, and sends a response back to the server. Once a payload is received, the worker processes it and sends a
response using the `respond` method.

## What's Next?

1. [PHP Workers](../php/worker.md) - Read more about PHP workers.
