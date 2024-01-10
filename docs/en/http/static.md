# HTTP — Serving static content

`Static` HTTP middleware serving static content using RoadRunner on the main HTTP plugin endpoint. Using this middleware
can slow down the overall performance by up to `~10%`, because RoadRunner has to check the path for each file request.

> **INFO**
> if there is no such file to serve, RR will redirect the request back to the PHP worker.

## Enable HTTP Middleware

To enable static content serving use the configuration inside the http section:

```yaml
version: "3"

http:
  # host and port separated by semicolon
  address: 127.0.0.1:44933
  middleware: [ "static" ] # <-- Add static to the list of the middleware
  # Settings for "static" middleware (docs: https://roadrunner.dev/docs/http-http/2023.x/en).
  static:
    dir: "."
    forbid: [ "" ]
    calculate_etag: false
    weak: false
    allow: [ ".txt", ".php" ]
    request:
      input: "custom-header"
    response:
      output: "output-header"
```

Where:

1. `dir`: path to the directory.
2. `forbid`: file extensions that should not be served.
3. `allow`: extensions that should be served (empty - serve all except forbidden). If extension is present in both (allow and forbidden) hashmaps - that is treated as we should forbid file extension.
4. `calculate_etag`: enable etag calculation for the static file.
5. `weak`: use a weak generator (/W), it uses only filename to generate a CRC32 sum. If false - all file content used to generate CRC32 sum.
6. `request/response`: custom headers for the static files.

To combine static content with other middleware, use the following sequence (static is always last in the line, then headers and gzip):

```yaml
version: "3"

http:
  # host and port separated by semicolon
  address: 127.0.0.1:44933
  middleware: [ "static", "headers", "gzip" ]
  # Settings for "headers" middleware (docs: https://roadrunner.dev/docs/http-http/2023.x/en).
  headers:
    cors:
      allowed_origin: "*"
      allowed_headers: "*"
      allowed_methods: "GET,POST,PUT,DELETE"
      allow_credentials: true
      exposed_headers: "Cache-Control,Content-Language,Content-Type,Expires,Last-Modified,Pragma"
      max_age: 600
  # Settings for "static" middleware (docs: https://roadrunner.dev/docs/http-http/2023.x/en).
  static:
    dir: "."
    forbid: [ "" ]
    calculate_etag: false
    weak: false
    allow: [ ".txt", ".php" ]
    request:
      input: "custom-header"
    response:
      output: "output-header"
```

## File server plugin

Fileserver plugin serves the static files. It works similar to the `static` HTTP middleware and has extended functionality.
Static HTTP middleware slows down request processing by `~10%` because RR has to check each request for the
corresponding file.
The file server plugin uses a different port and only serves static files.

## File server configuration

```yaml
fileserver:
  # File server address
  #
  # Error on empty
  address: 127.0.0.1:10101
  # Etag calculation. Request body CRC32.
  #
  # Default: false
  calculate_etag: true

  # Weak etag calculation
  #
  # Default: false
  weak: false

  # Enable body streaming for files more than 4KB
  #
  # Default: false
  stream_request_body: true

  serve:
    # HTTP prefix
    #
    # Error on empty
    - prefix: "/foo"

      # Directory to serve
      #
      # Default: "."
      root: "../../../tests"

      # When set to true, the server tries minimizing CPU usage by caching compressed files
      #
      # Default: false
      compress: false

      # Expiration duration for inactive file handlers. Units: seconds.
      #
      # Default: 10, use a negative value to disable it.
      cache_duration: 10

      # The value for the Cache-Control HTTP-header. Units: seconds
      #
      # Default: 10 seconds
      max_age: 10

      # Enable range requests
      # https://developer.mozilla.org/en-US/docs/Web/HTTP/Range_requests
      #
      # Default: false
      bytes_range: true

    - prefix: "/foo/bar"
      root: "../../../tests"
      compress: false
      cache_duration: 10s
      max_age: 10
      bytes_range: true
```
