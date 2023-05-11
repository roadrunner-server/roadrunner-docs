# KV Plugin — Redis Driver

Before configuring the Redis driver, please make sure that the Redis Server is installed and running.

> **Note**
> You can read more about this [in the documentation](https://redis.io/).

## Configuration

In the simplest case, when a full-fledged cluster or a fault-tolerant system is not required, we have one connection to
the Redis Server.

The complete redis driver configuration:

```yaml
version: "3"

kv:
  # User defined name of the storage.
  redis:
    # Required section.
    # Should be "redis" for the redis driver.
    driver: redis

    config:
      # Optional section.
      # By default, one connection will be specified with the "127.0.0.1:6379" value.
      addrs:
        - "127.0.0.1:6379"

      # Optional section.
      # Default: ""
      username: ""

      # Optional section.
      # Default: ""
      password: ""

      # Optional section.
      # Default: 0
      db: 0

      # Optional section.
      # Default: 0 (equivalent to the default value of 5 seconds)
      dial_timeout: 0

      # Optional section.
      # Default: 0 (equivalent to the default value of 3 retries)
      max_retries: 0

      # Optional section.
      # Default: 0 (equivalent to the default value of 8ms)
      min_retry_backoff: 0

      # Optional section.
      # Default: 0 (equivalent to the default value of 512ms)
      max_retry_backoff: 0

      # Optional section.
      # Default: 0 (equivalent to the default value of 10 connections per CPU).
      pool_size: 0

      # Optional section.
      # Default: 0 (do not use idle connections)
      min_idle_conns: 0

      # Optional section.
      # Default: 0 (do not close aged connections)
      max_conn_age: 0

      # Optional section.
      # Default: 0 (equivalent to the default value of 3s)
      read_timeout: 0

      # Optional section.
      # Default: 0 (equivalent to the value specified in the "read_timeout" section)
      write_timeout: 0

      # Optional section.
      # Default: 0 (equivalent to the value specified in the "read_timeout" + 1s)
      pool_timeout: 0

      # Optional section.
      # Default: 0 (equivalent to the default value of 5m)
      idle_timeout: 0

      # Optional section.
      # Default: 0 (equivalent to the default value of 1m)
      idle_check_freq: 0

      # Optional section.
      # Default: false
      read_only: false
```

## Options

Below is a more detailed description of each of the Redis-specific options:

### Addresses

`addrs`: An array of strings of connections to the Redis Server. Must contain at least one value of an existing
connection in the format of host or IP address and port, separated by a colon (`:`) character.

### Username and Password

`username`: Optional value containing the username credentials of the Redis connection. You can omit this field, or
specify an empty string if the username of the connection is not specified.

`password`: Optional value containing the password credentials of the Redis connection. You can omit this field, or
specify an empty string if the password of the connection is not specified.

### DB

`db`: An optional identifier for the database used in this connection to the Redis Server.

> **Note**
> Read more about databases section on the documentation page for the description of
> the [select command](https://redis.io/commands/select).

### Timeouts

`dial_timeout`: Server connection timeout. A value of `0` is equivalent to a timeout of 5 seconds (`5s`). After the
specified time has elapsed, if the connection has not been established, a connection error will occur.

Must be in the format of a "numeric value" + "time format suffix", like "`2h`" where suffixes means:

- `h` - the number of hours. For example `1h` means 1 hour.
- `m` - the number of minutes. For example `2m` means 2 minutes.
- `s` - the number of seconds. For example `3s` means 3 seconds.
- `ms` - the number of milliseconds. For example `4ms` means 4 milliseconds.

If no suffix is specified, the value will be interpreted as specified in nanoseconds. In most cases, this accuracy is
redundant and may not be true. For example `5` means 5 nanoseconds.

> **Note**
> All time intervals can be suffixed.


`read_timeout`: Timeout for socket reads. If reached, commands will fail with a timeout instead of blocking. Must be in
the format of a "numeric value" + "time format suffix". A value of `0` is equivalent to a timeout of 3 seconds (`3s`). A
value of `-1` disables timeout.

`write_timeout`: Timeout for socket writes. If reached, commands will fail with a timeout instead of blocking. A value
of `0` is equivalent of the value specified in the `read_timeout` section. If `read_timeout` value is not specified, a
value of 3 seconds (`3s`) will be used.

`pool_timeout`: Amount of time client waits for connection if all connections are busy before returning an error. A
value of `0` is equivalent of the value specified in the `read_timeout` + `1s`. If `read_timeout` value is not
specified, a value of 4 seconds (`4s`) will be used.

`idle_timeout`: Amount of time after which client closes idle connections. Must be in the format of a
"numeric value" + "time format suffix". A value of `0` is equivalent to a timeout of 5 minutes (`5m`). A value of `-1`
disables idle timeout check.

### Retries

`max_retries`: Maximum number of retries before giving up. Specifying `0` is equivalent to the default (`3` attempts).
If you need to specify an infinite number of connection attempts, specify the value `-1`.

`min_retry_backoff`: Minimum backoff between each retry. Must be in the format of a "numeric value" + "time format
suffix". A value of `0` is equivalent to a timeout of 8 milliseconds (`8ms`). A value of `-1` disables backoff.

`max_retry_backoff`: Maximum backoff between each retry. Must be in the format of a "numeric value" + "time format
suffix". A value of `0` is equivalent to a timeout of 512 milliseconds (`512ms`). A value of `-1` disables backoff.

### Pool Size

`pool_size`: Maximum number of RoadRunner socket connections. A value of `0` is equivalent to a `10` connections per
every CPU. Please note that specifying the value corresponds to the number of connections **per core**, so if you have 8
cores in your system, then setting the option to 2 you will get 16 connections.

### Other

`min_idle_conns`: Minimum number of idle connections which is useful when establishing new connection is slow. A value 
of 0 means no such idle connections. More details about the problem requiring the presence of this option available in 
the [corresponding issue](https://github.com/go-redis/redis/issues/772).

`max_conn_age`: Connection age at which client retires (closes) the connection. A value of `0` is equivalent to a 
disabling this option. In this case, aged connections will not be closed.

`idle_check_freq`: Frequency of idle checks made by idle connections reaper. Must be in the format of 
a "numeric value" + "time format suffix". A value of`0` is equivalent to a timeout of 1 minute (`1m`). A value of `-1` 
disables idle connections reaper. Note, that idle connections are still discarded by the client if `idle_timeout` is 
set.

`read_only`: An optional boolean value that enables or disables read-only mode. If `true` value is specified, the 
writing will be unavailable. Note that this option **not allowed** when working with Redis Sentinel.

These are all options available for all Redis connection types.

## Redis Cluster

In the case that you want to configure a [Redis Cluster](https://redis.io/topics/cluster-tutorial), then you can specify
additional options required only if you are organizing this type of server.

When creating a cluster, multiple connections are available to you. For example, you call such a
command `redis-cli --cluster create 127.0.0.1:6379 127.0.0.1:6380`, you should specify the appropriate set of
connections. In addition, when organizing a cluster, two additional options with algorithms for working with connections
will be available to you: `route_by_latency` and `route_randomly`.

```yaml
version: "3"

kv:
  redis:
    driver: redis
    config:
      addrs:
        - "127.0.0.1:6379"
        - "127.0.0.1:6380"

      # Optional section.
      # Default: false
      route_by_latency: false

      # Optional section.
      # Default: false
      route_randomly: false
```

Where new options means:

- `route_by_latency`: Allows routing read-only commands to the closest master
  or slave node. If this option is specified, the `read_only` configuration value
  will be automatically set to `true`.

- `route_randomly`: Allows routing read-only commands to the random master or
  slave node. If this option is specified, the `read_only` configuration value
  will be automatically set to `true`.

## Redis Sentinel

Redis Sentinel provides high availability for Redis. You can find more information
about [Sentinel on the documentation page](https://redis.io/topics/sentinel).

There are two additional options available for the Sentinel configuration: `master_name` and `sentinel_password`.

```yaml
version: "3"

kv:
  redis:
    driver: redis

    config:
      # Required section.
      master_name: ""

      # Optional section.
      # Default: "" (no password)
      sentinel_password: ""
```

Where Sentinel's options means:

- `master_name`: The name of the Sentinel's master in string format.

- `sentinel_password`: Sentinel password from "requirepass `password`"
  (if enabled) in Sentinel configuration.
