# Local (based on the boltdb) Driver

This type of driver is already supported by RoadRunner and requires no additional installation. It uses boltdb as the main job store. 
This driver should be used locally for testing or development. It can be used in production, but this type of driver can't handle huge loads.
huge load. The maximum RPS it can have is not more than 30-50.

Data in this driver is stored in the boltdb database file. You can't use the same file at the same time for the 2 pipelines or for the
for KV plugin and Jobs plugin. This is boltdb limitation on simultaneous access of 2 processes to the same file.

## Configuration:

```yaml
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
        # BoldDB file to create or DB to use
        # Default: "rr.db"
        file: "path/to/rr.db"

        # Optional section.
        # Default: 10
        priority: 10
      
        # Optional section.
        # Default: 1000
        prefetch: 1000
```

Below is a more detailed description of each of the boltdb options:
- `priority`: Queue default priority for each task pushed into this queue, if the priority value for these tasks has not been explicitly set.
  Lower value - higher priority. For example, we have 2 pipelines `pipe1` with priority 1 and `pipe10` with priority 10.
  Jobs from `pipe10` will be taken by workers only if all jobs from `pipe1` are processed.

- `prefetch`: A number of messages to receive from the local queue until ACK/NACK.

- `file`: boltdb database file to use. Might be a full path with file: `/foo/bar/rr1.db`. Default: `rr.db`.
