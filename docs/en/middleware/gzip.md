## Gzip middleware

## Documentation

- MDN [link](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding)

## Configuration

```yaml
version: '2.7'

http:
  address: 127.0.0.1:15389
  middleware: [ gzip ] 
  pool:
    num_workers: 10
    allocate_timeout: 60s
    destroy_timeout: 60s
```