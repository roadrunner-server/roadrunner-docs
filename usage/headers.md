# Headers and CORS
RoadRunner can automatically setup request/response headers and control CORS for your application.

### CORS
To enable CORS headers add the following section to your configuration.

```yaml
headers:
  # Middleware to handle CORS requests, https://www.w3.org/TR/cors/
  cors:
    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin
    allowedOrigin: "*"

    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Headers
    allowedHeaders: "*"

    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Methods
    allowedMethods: "GET,POST,PUT,DELETE"

    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials
    allowCredentials: true

    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Expose-Headers
    exposedHeaders: "Cache-Control,Content-Language,Content-Type,Expires,Last-Modified,Pragma"

    # Max allowed age in seconds
    maxAge: 600
```

### Custom headers for Response or Request
You can control additional headers to be set for outgoing responses and headers to be added to the request send to your application.
```yaml
headers:
  # Automatically add headers to every request passed to PHP.
  request:
    "Example-Request-Header": "Value"

  # Automatically add headers to every response.
  response:
    "X-Powered-By": "RoadRunner"
```
