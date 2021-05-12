# Serving static content
It is possible to serve static content using RoadRunner.

## Enable HTTP Middleware
To enable static content serving use the configuration inside the http section:

```yaml
http:
  # host and port separated by semicolon
  address: 127.0.0.1:44933
  # ...
    static:
      dir: "."
      pattern: "/tests/"
      forbid: [""]
      allow: [".txt", ".php"]
      calculate_etag: false
      weak: false
      request:
        input: "custom-header"
      response:
        output: "output-header"
```
Where:
1. `dir`: path to the directory.
2. `pattern`: http path to serve witin the directory. For example: dir - `/home/user/`, pattern - `/static/` will point to the `static` dir in the `/home/user/` (`/home/user/static`).
3. `forbid`: file extensions that should not be served.
4. `allow`: file extensions which should be served (empty - serve all except forbidden).
5. `calculate_etag`: turn on etag computation for the static file.
6. `weak`: use a weak generator (/W).
7. `request/response`: custom headers for the static files.  


To combine static content with other middleware, use the following sequence (static will always be the last in the row, file server will apply headers and gzip plugins):

```yaml
http:
  # host and port separated by semicolon
  address: 127.0.0.1:44933
  # ...
  middleware: [ "headers", "gzip" ]
  # ...
  headers:
    # ...
    static:
      dir: "."
      pattern: "/tests/"
      forbid: [""]
      allow: [".txt", ".php"]
      calculate_etag: false
      weak: false
      request:
        input: "custom-header"
      response:
        output: "output-header"
```
