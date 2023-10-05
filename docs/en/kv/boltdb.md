# KV Plugin — Boltdb Driver

This type of driver is already supported by the RoadRunner and does not require any additional installations.

## Configuration

The complete boltdb driver configuration:

```yaml
version: "3"

kv:
  # User defined name of the storage.
  boltdb:
    # Required section.
    # Should be "boltdb" for the boltdb driver.
    driver: boltdb

    config:
      # Optional section.
      # Default: "rr.db"
      file: "./rr.db"

      # Optional section.
      # Default: 0777
      permissions: 0777

      # Optional section.
      # Default: "rr"
      bucket: "rr"

      # Optional section.
      # Default: 60
      interval: 60
```

## Options

Below is a more detailed description of the various boltdb options.:

### File

`file`: Database file path name. In the case that such a file does not exist, RoadRunner will create this file on its
own at startup. Note that this must be an existing directory, otherwise a "The system cannot find the path specified"
error will occur, indicating that the full database pathname is invalid. Might be a full path with
file: `/foo/bar/rr1.db`. Default: `rr.db`.

### Permissions

`permissions`: The file permissions in UNIX format of the database file, set at the time of its creation. If the file
already exists, the permissions will not be changed.

### Bucket

`bucket`: The bucket name. You can create several boltdb connections by specifying different buckets and in this case
the data stored in one bucket will
not intersect with the data stored in the other, even if the database file and other settings are completely
identical.

### Interval

`interval`: The interval (in seconds) between checks for the lifetime of the
value in the cache. The meaning and behavior is similar to that used in the
case of the memory driver.
