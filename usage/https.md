# HTTPS and HTTP/2

You can enable HTTPS and HTTP2 support by adding `ssl` section into `http` config.

```yaml
http:
  # http host to listen.
  address:   0.0.0.0:8080

  # HTTP service provides HTTP2 transport
  http2:
    # enable HTTP/2, only with TSL, enabled by default
    enabled: true

    # max transfer channels, default 128
    maxConcurrentStreams: 128

  ssl:
    # force redirect to https connection
    redirect: true

    # custom https port (default 443)
    port:     443

    # ssl cert
    cert:     server.crt

    # ssl private key
    key:      server.key
```

### Redirecting HTTP to HTTPS

To enable automatic redirect from `http://` to `https://` set `redirect` option to `true` (disabled by default).

### HTTP/2 Push Resources

RoadRunner support (HTTP/2 push)[https://en.wikipedia.org/wiki/HTTP/2_Server_Push] via virtual headers provided by PHP response.

```php
return $response->withAddedHeader('http2-push', '/test.js');
```

Note that the resources path must be related to the public application directory and must include `/` at the beginning.

> Please note, HTTP2 push only works under HTTPS with `static` service enabled.

