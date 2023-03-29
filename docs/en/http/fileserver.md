# File server plugin

Fileserver plugin serves the static files. It works similar to the `static` HTTP middleware and has extended
functionality.
Static HTTP middleware slows down request processing by `~10%` because RR has to check each request for the
corresponding file.
The file server plugin uses a different port and only serves static files.

## Configuration

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