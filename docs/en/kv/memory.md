# KV Plugin — Memory Driver

This type of driver is already supported by the RoadRunner and does not require any additional installations.

> **Warning**
> This type of storage, all data is contained in memory and will be destroyed when the RoadRunner Server is restarted.
> If you need persistent storage without additional dependencies, then it is recommended to use the boltdb driver.

## Configuration

The complete memory driver configuration:

```yaml
version: "3"

kv:
  # User defined name of the storage.
  memory:
    # Required section.
    # Should be "memory" for the memory driver.
    driver: memory

    config:
      # Optional section.
      # Default: 60
      interval: 60
```

## Options

Below is a more detailed description of each of the memory-specific options:

### Interval

`interval`: The interval (in seconds) between checks for the lifetime of the value in the cache. For large values of
the interval, the cache item will be checked less often for expiration of its lifetime. It is recommended to use large
values only in cases when the cache is used without expiration values, or in cases when this value is not critical to
the architecture of your application. Note that the lower this value, the higher the load on the system.
