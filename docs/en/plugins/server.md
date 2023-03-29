# Server plugin

Server plugin works with worker creation. Each plugin that uses workers, could easily, with 1 method, request the worker pool or single worker.
All created workers will automatically inherit all RRs functionality like: supervising, state machine, command handler, etc.

## PHP Client

- [Roadrunner Worker](https://github.com/spiral/roadrunner-worker)

## Configuration

```yaml
# Application server settings (docs: https://roadrunner.dev/docs/php-worker)
server:
  on_init:
    # Command to execute before the main server's command
    #
    # This option is required if using on_init
    command: "any php or script here"

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

  # Worker relay can be: "pipes", TCP (eg.: tcp://127.0.0.1:6002), or socket (eg.: unix:///var/run/rr.sock).
  #
  # Default: "pipes"
  relay: pipes

  # Timeout for relay connection establishing (only for socket and TCP port relay).
  #
  # Default: 60s
  relay_timeout: 60s
```