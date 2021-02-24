# Serving static content
It is possible to serve static content using RoadRunner.

## Enable HTTP Middleware
To enable static content serving enable `static` middleware, and it's config inside the http section:

```yaml
http:
  # host and port separated by semicolon
  address: 127.0.0.1:44933
  # ...
  middleware: [ "static" ]
  # ...
  static:
    dir: "tests"
    forbid: [ ] # file extensions to forbid
    request:
      "input": "custom-header"
    response:
      "output": "output-header"
```

To combine static content with other middleware use the following sequence:

```yaml
http:
  # host and port separated by semicolon
  address: 127.0.0.1:44933
  # ...
  middleware: [ "static", "headers", "gzip" ]
  # ...
  headers:
    # ...
  static:
    dir: "tests"
    forbid: [ ] # file extensions to forbid
    request:
      "input": "custom-header"
    response:
      "output": "output-header"
```