## PHP Client

- [Roadrunner GRPC](https://github.com/spiral/roadrunner-grpc)

## Compiling proto sample:

```
protoc --plugin=protoc-gen-php-grpc --php_out=<OUTPUT DIRECTORY> --php-grpc_out=<OUTPUT DIRECTORY> simple.proto
```
- `protoc-gen-php-grpc` plugin can be downloaded from the `roadrunner` [releases page](https://github.com/roadrunner-server/roadrunner/releases).

## Configuration

```yaml
version: "3"

grpc:
  # GRPC address to listen
  #
  # This option is required
  listen: "tcp://127.0.0.1:9001"

  # Proto file to use, multiply files supported [SINCE 2.6]
  #
  # This option is required
  proto:
    - "first.proto"
    - "second.proto"

  # GRPC TLS configuration
  #
  # This section is optional
  tls:
    # Path to the key file
    #
    # This option is required
    key: ""

    # Path to the certificate
    #
    # This option is required
    cert: ""

    # Path to the CA certificate, defines the set of root certificate authorities that servers use if required to verify a client certificate. Used with the `client_auth_type` option.
    #
    # This option is optional
    root_ca: ""

    # Client auth type.
    #
    # This option is optional. Default value: no_client_certs. Possible values: request_client_cert, require_any_client_cert, verify_client_cert_if_given, require_and_verify_client_cert, no_client_certs
    client_auth_type: no_client_certs

  # Maximum send message size
  #
  # This option is optional. Default value: 50 (MB)
  max_send_msg_size: 50

  # Maximum receive message size
  #
  # This option is optional. Default value: 50 (MB)
  max_recv_msg_size: 50

  # MaxConnectionIdle is a duration for the amount of time after which an
  #	idle connection would be closed by sending a GoAway. Idleness duration is
  #	defined since the most recent time the number of outstanding RPCs became
  #	zero or the connection establishment.
  #
  # This option is optional. Default value: infinity.
  max_connection_idle: 0s

  # MaxConnectionAge is a duration for the maximum amount of time a
  #	connection may exist before it will be closed by sending a GoAway. A
  #	random jitter of +/-10% will be added to MaxConnectionAge to spread out
  #	connection storms.
  #
  # This option is optional. Default value: infinity.
  max_connection_age: 0s

  # MaxConnectionAgeGrace is an additive period after MaxConnectionAge after
  #	which the connection will be forcibly closed.
  max_connection_age_grace: 0s8h

  # MaxConnectionAgeGrace is an additive period after MaxConnectionAge after
  #	which the connection will be forcibly closed.
  #
  # This option is optional: Default value: 10
  max_concurrent_streams: 10

  # After a duration of this time if the server doesn't see any activity it
  #	pings the client to see if the transport is still alive.
  #	If set below 1s, a minimum value of 1s will be used instead.
  #
  # This option is optional. Default value: 2h
  ping_time: 1s

  # After having pinged for keepalive check, the server waits for a duration
  #	of Timeout and if no activity is seen even after that the connection is
  #	closed.
  #
  # This option is optional. Default value: 20s
  timeout: 200s

  # Usual workers pool configuration
  pool:
    # Debug mode for the pool. In this mode, pool will not pre-allocate the worker. Worker (only 1, num_workers ignored) will be allocated right after the request arrived.
    #
    # Default: false
    debug: false

    # Override server's command
    #
    # Default: empty
    command: "php my-super-app.php"

    # How many worker processes will be started. Zero (or nothing) means the number of logical CPUs.
    #
    # Default: 0
    num_workers: 0

    # Maximal count of worker executions. Zero (or nothing) means no limit.
    #
    # Default: 0
    max_jobs: 0

    # Timeout for worker allocation. Zero means 60s.
    #
    # Default: 60s
    allocate_timeout: 60s

    # Timeout for the reset timeout. Zero means 60s.
    #
    # Default: 60s
    reset_timeout: 60s

    # Timeout for worker destroying before process killing. Zero means 60s.
    #
    # Default: 60s
    destroy_timeout: 60s
```

## Metrics
- [link](../lab/metrics.md)

Example of grafana dashboard is available [here](https://github.com/roadrunner-server/roadrunner/blob/master/dashboards/grpc_dashboard.json).

## Minimal dependencies:

1. `Server` plugin for the workers pool.
2. `Logger` plugin to show log messages.
3. `Config` plugin to read and populate plugin's configuration.

## mTLS
To enable [mTLS](https://www.cloudflare.com/en-gb/learning/access-management/what-is-mutual-tls/) use the following configuration:

```yaml
version: "3"

grpc:
  listen: "tcp://localhost:9001"

  proto:
      - "first.proto"
      - "second.proto"

  tls:
    key: "server-key.pem"
    cert: "server-cert.pem"
    root_ca: "rootCA.pem"
    client_auth_type: request_client_cert
```

Options for the `client_auth_type` are:
- `request_client_cert`
- `require_any_client_cert`
- `verify_client_cert_if_given`
- `require_and_verify_client_cert`
- `no_client_certs`


## Common issues:
1. Registering two services with the same name is not allowed. GRPC server will panic after that.
