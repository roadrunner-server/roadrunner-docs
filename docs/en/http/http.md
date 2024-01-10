# HTTP Plugin

HTTP plugin is used to pass `HTTP`/`HTTPS`/`fCGI`/`HTTP2(h2c)`/`HTTP3` requests to the PHP worker.

## Configuration reference

```yaml .rr.yaml
version: "3"

# HTTP plugin settings.
http:
  # Host and port to listen on (eg.: `127.0.0.1:8080`).
  #
  # This option is required.
  address: 127.0.0.1:8080

  # override http error code for the internal RR errors
  #
  # Default: 500
  internal_error_code: 505

  # HTTP access logs
  #
  # Default: false
  access_logs: false

  # Maximal incoming request size in megabytes. Zero means no limit.
  #
  # Default: 0
  max_request_size: 256

  # Send raw body (unescaped) to the PHP worker for the application/x-www-form-urlencoded content type
  #
  # Optional, default: false
  raw_body: false

  # Middlewares for the http plugin, order is important. Allowed values is: "headers", "gzip", "static", "sendfile",  [SINCE 2.6] -> "new_relic", [SINCE 2.6] -> "http_metrics", [SINCE 2.7] -> "cache"
  #
  # Default value: []
  middleware: [ "headers", "gzip" ]

  # Allow incoming requests only from the following subnets (https://en.wikipedia.org/wiki/Reserved_IP_addresses).
  #
  # Default: ["10.0.0.0/8", "127.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16",  "::1/128", "fc00::/7", "fe80::/10"]
  trusted_subnets: [
    "10.0.0.0/8",
    "127.0.0.0/8",
    "172.16.0.0/12",
    "192.168.0.0/16",
    "::1/128",
    "fc00::/7",
    "fe80::/10",
  ]

  # File uploading settings.
  uploads:
    # Directory for file uploads. Empty value means to use $TEMP based on your OS.
    #
    # Default: ""
    dir: "/tmp"

    # Deny files with the following extensions to upload.
    #
    # Default: [".php", ".exe", ".bat"]
    forbid: [ ".php", ".exe", ".bat", ".sh" ]

    # [SINCE 2.6] Allow files with the following extensions to upload
    #
    # Default: empty
    allow: [ ".html", ".aaa" ]

  # Settings for "headers" middleware (docs: https://roadrunner.dev/docs/http-http/2023.x/en).
  headers:
    # Allows to control CORS headers. Additional headers "Vary: Origin", "Vary: Access-Control-Request-Method",
    # "Vary: Access-Control-Request-Headers" will be added to the server responses. Drop this section for this
    # feature disabling.
    cors:
      # Controls "Access-Control-Allow-Origin" header value (docs: https://mzl.la/2OgD4Qf).
      #
      # Default: ""
      allowed_origin: "*"

      # Controls "Access-Control-Allow-Headers" header value (docs: https://mzl.la/2OzDVvk).
      #
      # Default: ""
      allowed_headers: "*"

      # Controls "Access-Control-Allow-Methods" header value (docs: https://mzl.la/3lbwyXf).
      #
      # Default: ""
      allowed_methods: "GET,POST,PUT,DELETE"

      # Controls "Access-Control-Allow-Credentials" header value (docs: https://mzl.la/3ekJGaY).
      #
      # Default: false
      allow_credentials: true

      # Controls "Access-Control-Expose-Headers" header value (docs: https://mzl.la/3qAqgkF).
      #
      # Default: ""
      exposed_headers: "Cache-Control,Content-Language,Content-Type,Expires,Last-Modified,Pragma"

      # Controls "Access-Control-Max-Age" header value in seconds (docs: https://mzl.la/2PCSdvt).
      #
      # Default: 0
      max_age: 600

    # Automatically add headers to every request passed to PHP.
    #
    # Default: <empty map>
    request:
      input: "custom-header"

    # Automatically add headers to every response.
    #
    # Default: <empty map>
    response:
      X-Powered-By: "RoadRunner"

  # Settings for "static" middleware (docs: https://roadrunner.dev/docs/http-http/2023.x/en).
  static:
    # Path to the directory to serve
    #
    # Default: "." (current)
    dir: "."

    # File patterns to forbid
    #
    # Default: empty
    forbid: [ "" ]

    # Etag calculation (base on the body CRC32)
    #
    # Default: false
    calculate_etag: false

    # Weak etag calculation (based only on the content-length CRC32)
    #
    # Default: false
    weak: false

    # Patterns to allow
    #
    # Default: empty
    allow: [ ".txt", ".php" ]

    # Request headers
    #
    # Default: empty
    request:
      input: "custom-header"

    # Response headers
    #
    # Default: empty
    response:
      output: "output-header"

  # Workers pool settings.
  pool:
    # Debug mode for the pool. In this mode, pool will not pre-allocate the worker. Worker (only 1, num_workers ignored) will be allocated right after the request arrived.
    #
    # Default: false
    debug: false

    # Override server's command
    #
    # Default: empty
    command: "php my-super-app.php"

    # How many worker processes will be started. Zero (or nothing) means the number of logical CPUs.
    #
    # Default: 0
    num_workers: 0

    # Maximal count of worker executions. Zero (or nothing) means no limit.
    #
    # Default: 0
    max_jobs: 0

    # Timeout for worker allocation. Zero means 60s.
    #
    # Default: 60s
    allocate_timeout: 60s

    # Timeout for the reset timeout. Zero means 60s.
    #
    # Default: 60s
    reset_timeout: 60s

    # Timeout for worker destroying before process killing. Zero means 60s.
    #
    # Default: 60s
    destroy_timeout: 60s

    # Supervisor is used to control http workers (previous name was "limit", video: https://www.youtube.com/watch?v=NdrlZhyFqyQ).
    # "Soft" limits will not interrupt current request processing. "Hard"
    # limit on the contrary - interrupts the execution of the request.
    supervisor:
      # How often to check the state of the workers.
      #
      # Default: 1s
      watch_tick: 1s

      # Maximum time worker is allowed to live (soft limit). Zero means no limit.
      #
      # Default: 0s
      ttl: 0s

      # How long worker can spend in IDLE mode after first using (soft limit). Zero means no limit.
      #
      # Default: 0s
      idle_ttl: 10s

      # Maximal worker memory usage in megabytes (soft limit). Zero means no limit.
      #
      # Default: 0
      max_worker_memory: 128

      # Maximal job lifetime (hard limit). Zero means no limit.
      #
      # Default: 0s
      exec_ttl: 60s

  # SSL (Secure Sockets Layer) (TLS) settings (docs: https://roadrunner.dev/docs/http-http/2023.x/en).
  ssl:
    # Host and port to listen on (eg.: `127.0.0.1:443`).
    #
    # Default: ":443"
    address: "127.0.0.1:443"

    # Use ACME certificates provider (Let's encrypt)
    acme:
      # Directory to use as a certificate/pk, account info storage
      #
      # Optional. Default: rr_cache
      certs_dir: rr_le_certs

      # User email
      #
      # Used to create LE account. Mandatory. Error on empty.
      email: you-email-here@email

      # Alternate port for the http challenge. Challenge traffic should be redirected to this port if overridden.
      #
      # Optional. Default: 80
      alt_http_port: 80


      # Alternate port for the tls-alpn-01 challenge. Challenge traffic should be redirected to this port if overridden.
      #
      # Optional. Default: 443.
      alt_tlsalpn_port: 443

      # Challenge types
      #
      # Optional. Default: http-01. Possible values: http-01, tlsalpn-01
      challenge_type: http-01

      # Use production or staging endpoint. NOTE, try to use staging endpoint to make sure, that everything works correctly.
      #
      # Optional, but for production should be set to true. Default: false
      use_production_endpoint: true

      # List of your domains to obtain certificates
      #
      # Mandatory. Error on empty.
      domains: [
        "your-cool-domain.here",
        "your-second-domain.here"
      ]

    # Automatic redirect from http:// to https:// schema.
    #
    # Default: false
    redirect: true

    # Path to the cert file. This option is required for SSL working.
    #
    # This option is required.
    cert: /ssl/server.crt

    # Path to the cert key file.
    #
    # This option is required.
    key: /ssl/server.key

    # Path to the root certificate authority file.
    #
    # This option is optional (required for the mTLS).
    root_ca: /ssl/root.crt

    # Client auth type (mTLS)
    #
    # This option is optional. Default value: no_client_certs. Possible values: request_client_cert, require_any_client_cert, verify_client_cert_if_given, require_and_verify_client_cert, no_client_certs
    client_auth_type: no_client_certs

  # FastCGI frontend support.
  fcgi:
    # FastCGI connection DSN. Supported TCP and Unix sockets. An empty value disables this.
    #
    # Default: ""
    address: tcp://0.0.0.0:7921

  # HTTP/2 settings.
  http2:
    # HTTP/2 over non-encrypted TCP connection using H2C.
    #
    # Default: false
    h2c: false

    # Maximal concurrent streams count.
    #
    # Default: 128
    max_concurrent_streams: 128
```

## HTTPS

You can enable HTTPS support by adding `ssl` section into `http` config.

```yaml
version: "3"

http:
  address: 127.0.0.1:8080

  ssl:
    # host and port separated by semicolon (default :443)
    address: :8892
    redirect: false
    cert: fixtures/server.crt
    key: fixtures/server.key
    root_ca: root.crt
```

### Let's Encrypt

RR can automatically obtain TLS certificates for your domain. The folder with your certs might be moved between servers,
RR will check the `certs_dir` and obtain a new certificate if the old one is above to expire. 

RR will track your certificate's expiration date and refresh it automatically.

```yaml .rr.yaml
version: "3"

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

### mTLS

To enable [mTLS](https://www.cloudflare.com/en-gb/learning/access-management/what-is-mutual-tls/) use the following
configuration:

```yaml .rr.yaml
http:
  pool:
    num_workers: 1
    max_jobs: 0
    allocate_timeout: 60s
    destroy_timeout: 60s
  ssl:
    address: :8895
    key: "server-key.pem"
    cert: "server-cert.pem"
    root_ca: "rootCA.pem"
    client_auth_type: require_and_verify_client_cert 
```

**Options for the `client_auth_type` are:**

- `request_client_cert`
- `require_any_client_cert`
- `verify_client_cert_if_given`
- `require_and_verify_client_cert`
- `no_client_certs`

### Redirecting HTTP to HTTPS

To enable an automatic redirect from `http://` to `https://` set `redirect` option to `true` (disabled by default).

### Root certificate authority support

Root CA supported by the option in `.rr.yaml`

```yaml
version: "3"

http:
  ssl:
    root_ca: root.crt
```

## HTTP/2

You can enable HTTP2 support by adding `http2` section into `http` config.

```yaml
version: "3"

http:
  address: 127.0.0.1:8080

  http2:
    h2c: false
    max_concurrent_streams: 128
```

### Upgrade connection from `http1.1` to `h2c` [`v2.10.2`]

Connection might be upgraded from the `http/1.1`
to `h2c`: [rfc7540](https://datatracker.ietf.org/doc/html/rfc7540#section-3.4)

**Headers, which should be sent to upgrade connection:**

1. `Upgrade`: `h2c`
2. `Connection`: `HTTP2-Settings`
3. `Connection`: `Upgrade`
4. `HTTP2-Settings`: `AAMAAABkAARAAAAAAAIAAAAA` [RFC](https://datatracker.ietf.org/doc/html/rfc7540#section-3.2.1)

### HTTP/2 Push Resources

RoadRunner support [HTTP/2 push](https://en.wikipedia.org/wiki/HTTP/2_Server_Push) via virtual headers provided by PHP
response.

```php
return $response->withAddedHeader('http2-push', '/test.js');
```

Note that the path of the resource must be related to the public application directory and must include `/` at the
beginning.

> **Note**
> HTTP2 push only works under HTTPS with `static` service enabled.

### H2C

You can enable HTTP/2 support over non-encrypted TCP connection using H2C:

```yaml
version: "3"

http:
  http2.h2c: true
```

## FastCGI

There is FastCGI frontend support inside the HTTP module, you can enable it (disabled by default):

```yaml
version: "3"

http:
  fcgi:
    # FastCGI connection DSN. Supported TCP and Unix sockets.
    address: tcp://0.0.0.0:6920
```

## HTTP/3
HTTP3 support is experimental and might be changed in the future. Docs are available in the [experimental](../experimental/experimental.md) section.

## Overriding HTTP default error code

```yaml
version: "3"

http:
  # override http error code for the internal RR errors (default 500)
  internal_error_code: 505
```

`http.internal_error_code` code is used for the `SoftJob`, allocation, all kinds of TTL, Network, errors. But for
example, for the load balancer might be better to use a different code. So, you may override the default one.

## Middleware order

Since all middleware are independent, they can remove/update headers set by
the [previous one](https://github.com/roadrunner-server/roadrunner/issues/1501).

**Note that the request (imagine) comes from the right:**

```yaml
http:
  middleware: # RESPONSE FROM HERE --> [ "static", "gzip", "sendfile" ] # <-- REQUEST COMES FROM HERE
```

So in this case the request gets into the `sendfile` middleware, then `gzip` and `static`. And vice versa from the
response.

## Request queues

RR has an internal queue for requests. The `allocate_timeout` is used to assign a worker to the request. 
If your worker works for 1 minute for example, but `allocate_timeout` is equal to 30 seconds, after this timeout, RR will start rejecting the first request in queue. 
Then +30s for the second, and so on and so forth.
