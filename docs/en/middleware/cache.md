## Cache (RFC7234) middleware [In progress]

Cache middleware implements http-caching RFC 7234 (not fully yet).  
It handles the following headers:

- `Cache-Control`
- `max-age`

**Responses**:

- `Age`

**HTTP codes**:

- `OK (200)`

**Available backends**:

- `memory`

**Available methods**:

- `GET`

## Configuration

```yaml
version: "2.7"

http:
  address: 127.0.0.1:44933
  middleware: ["cache"]
  # ...
  cache:
    driver: memory
    cache_methods: ["GET", "HEAD", "POST"] # only GET supported at the moment
    config: {}
```

For the worker sample and other docs, please, refer to the http plugin.
