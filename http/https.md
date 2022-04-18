# HTTPS and HTTP/2

You can enable HTTPS and HTTP2 support by adding `ssl` section into `http` config.

```yaml
version: "2.7"

http:
  # host and port separated by semicolon
  address: 127.0.0.1:8080
 
  ssl:
    # host and port separated by semicolon (default :443)
    address: :8892
    redirect: false
    cert: fixtures/server.crt
    key: fixtures/server.key
    root_ca: root.crt
  
  # optional support for http2  
  http2:
    h2c: false
    max_concurrent_streams: 128
```

### Let's Encrypt

RR starting from the `v2.5.0` can automatically obtain TLS certificates for your domain. The folder with your certs might be moved between servers, RR will check the `certs_dir` and obtain a new certificate if the old one is above to expire.
RR will track your certificate's expiration date and refresh it automatically.

### Configuration

```yaml
version: "2.7"

http:
  # other HTTP sections are omitted 
  # .......
  
  ssl:
    address: '0.0.0.0:443'
    # ACME section
    #
    # TLS provider
    acme:
      # directory to store your certificate and key from the LE
      #
      # Default: rr_cache_dir
      certs_dir: rr_le_certs
      # Your email
      #
      # Mandatory. Error on empty.
      email: you-email-here@email
      # Alternate port for the HTTP challenge (make sure, that you redirected traffic to the specified port from 80)
      #
      # Default: 80
      alt_http_port: 80
      # Alternate port for the TLS-ALPN challenge (make sure, that you redirected traffic to the specified port from 443)
      #
      # Default: 443
      alt_tlsalpn_port: 443
      # Challenge type to use
      #
      # Default: http-01
      challenge_type: http-01
      # Use staging or production endpoint
      #
      # Would be a good practice to test your setup, before obtaining a real certificate
      use_production_endpoint: false
      # List of your domains
      #
      # Mandatory. Error on empty
      domains:
        - your-cool-domains.here
        
 # other HTTP sections are omitted
 # ........
```

### Redirecting HTTP to HTTPS

To enable an automatic redirect from `http://` to `https://` set `redirect` option to `true` (disabled by default).

### HTTP/2 Push Resources

RoadRunner support [HTTP/2 push](https://en.wikipedia.org/wiki/HTTP/2_Server_Push) via virtual headers provided by PHP
response.

```php
return $response->withAddedHeader('http2-push', '/test.js');
```

Note that the path of the resource must be related to the public application directory and must include `/` at the
beginning.

> Please note, HTTP2 push only works under HTTPS with `static` service enabled.

## H2C

You can enable HTTP/2 support over non-encrypted TCP connection using H2C:

```yaml
version: "2.7"

http:
  http2.h2c: true
```

### FastCGI

There is FastCGI frontend support inside the HTTP module, you can enable it (disabled by default):

```yaml
version: "2.7"

http:
  # HTTP service provides FastCGI as frontend
  fcgi:
    # FastCGI connection DSN. Supported TCP and Unix sockets.
    address: tcp://0.0.0.0:6920
```

### Root certificate authority support

Root CA supported by the option in .rr.yaml

```yaml
version: "2.7"

http:
  ssl:
    root_ca: root.crt
```