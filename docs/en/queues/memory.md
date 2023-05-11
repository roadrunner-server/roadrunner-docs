# Jobs — Memory Driver

This type of driver is already supported by the RoadRunner and does not require any additional installations.

Note that using this type of queue driver, all data is in memory and will be destroyed when the RoadRunner Server is
restarted. If you need persistent queue, then it is recommended to use alternative drivers: `amqp`, `beanstalk`
or `sqs`.

> **Warning**
> This driver cannot hold more than **1000 tasks** with delay at the same time (RR limitation)

## Configuration

```yaml .rr.yaml
version: "3"

jobs:
  pipelines:
    # User defined name of the queue.
    example:
      # Required section.
      # Should be "memory" for the in-memory driver.
      driver: memory

      config: # NEW in 2.7
        # Optional section.
        # Default: 10
        priority: 10

        # Optional section.
        # Default: 10
        prefetch: 10
```

## Configuration options

**Here is a detailed description of each of the in-memory-specific options:**

### Priority

`priority` - Queue default priority for each task pushed into this queue if the priority value for these tasks was not
explicitly set. Lower value - higher priority.

**For example, we have 2 pipelines "pipe1" with priority 1 and "pipe10" with priority 10. Jobs from "pipe10" will be
taken by workers only if all the jobs from "pipe1" are handled.**

### Prefetch

`prefetch` - A local buffer between the PQ (priority queue) and driver. If the PQ size is set to 100 and prefetch to
100000, you'll be able to push up to prefetch number of jobs even if PQ is full.
