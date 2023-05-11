# KV Plugin — Memcached Driver

Before configuring the Memcached driver, please make sure that the Memcached Server is installed and running. You can
read more about this [in the documentation](https://memcached.org/).

## Configuration

The complete memcached driver configuration:

```yaml
version: "3"

kv:
  # User defined name of the storage.
  memcached:
    # Required section.
    # Should be "memcached" for the memcached driver.
    driver: memcached
    config:
      # Optional section.
      # Default: "127.0.0.1:11211"
      addr: "127.0.0.1:11211"
```

## Options

Below is a more detailed description of each of the memcached-specific options:

### Addr

`addr`: String of memcached connection in format "`[HOST]:[PORT]`".

In the case that there are several memcached servers, then the list of connections can be listed in an array format, for
example:

```yaml
version: "3"

kv:
  memcached:
    driver: memcached
    config:
      addr: [ "127.0.0.1:11211", "127.0.0.1:11222" ]
```