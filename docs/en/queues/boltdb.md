# Jobs — Local (based on the boltdb) Driver

This type of driver is already supported by RoadRunner and requires no additional installation. It uses boltdb as the
main job store. This driver should be used locally for testing or development. It can be used in production, but this
type of driver can't handle huge loads. huge load. The maximum RPS it can have is not more than 30-50.

Data in this driver is stored in the boltdb database file. You can't use the same file at the same time for the 2
pipelines or for the for KV plugin and Jobs plugin. This is a boltdb limitation on simultaneous access of 2 processes to
the same file.

## Configuration

```yaml .rr.yaml
version: "3"

boltdb:
  permissions: 0777

jobs:
  pipelines:
    # User defined name of the queue.
    example:
      # Required section.
      # Should be "boltdb" for the local driver.
      driver: boltdb

      config: # NEW in 2.7
        # Number of jobs to prefetch from the driver.
        #
        # Default: 100_000.
        prefetch: 10000

        # Pipeline priority
        #
        # If the job has priority set to 0, it will inherit the pipeline's priority. Default: 10.
        priority: 10

        # BoldDB file to create or DB to use
        #
        # Default: "rr.db"
        file: "path/to/rr.db"

        # Permissions for the boltdb database file
        #
        # This option is optional. Default: 0755
        permissions: 0755
```

## Configuration options

**Here is a detailed description of each of the boltdb-specific options:**

### Priority

`priority` - Queue default priority for each task pushed into this queue, if the priority value for these tasks has not
been explicitly set.

Lower value - higher priority. For example, we have 2 pipelines `pipe1` with priority 1 and `pipe10` with priority 10.
Jobs from `pipe10` will be taken by workers only if all jobs from `pipe1` are processed.

### Prefetch

`prefetch` - A number of messages to receive from the local queue until ACK/NACK.

### File

`file` - boltdb database file to use. Might be a full path with file: `/foo/bar/rr1.db`. 

Default: `rr.db`.

### Permissions

`permissions` - Permissions for the boltdb database file. Default: `0755`.
