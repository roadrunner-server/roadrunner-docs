# HTTP — Gzip middleware

The gzip middleware is used to support the `accept-encodin: gzip` header and to compress and decompress the contents of the
outgoing/incoming requests.

## Documentation

- MDN [link](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding)

## Configuration

```yaml
version: "3"

http:
  address: 127.0.0.1:15389
  middleware: [ gzip ]
  pool:
    num_workers: 10
    allocate_timeout: 60s
    destroy_timeout: 60s
```

Gzip middleware supports OpenTelemetry headers propagation.
