# Embedded Monitoring
RoadRunner is capable of monitoring your application and run soft reset (between requests) if necessary.

## Configuration
Edit your `.rr` file to specify limits for your application:

```yaml
# monitors rr server(s)
limit:
  services.http:
      maxMemory: 100 # maximum allowed memory consumption per worker (soft)
      TTL: 0  # maximum time to live for the worker (soft)
      idleTTL: 0  # maximum allowed amount of time worker can spend in idle before being removed (for weak db connections, soft)
      execTTL: 60 # max_execution_time (process will be killed)
```