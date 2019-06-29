# Headers service

### CORS

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

### Customer headers for Response or Request

```yaml
headers:
  # Automatically add headers to every request passed to PHP.
  request:
    "Example-Request-Header": "Value"

  # Automatically add headers to every response.
  response:
    "X-Powered-By": "RoadRunner"
```
