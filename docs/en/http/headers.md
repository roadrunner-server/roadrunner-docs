# HTTP — Headers and CORS

Headers middleware is used to set up request/response headers and control CORS for your application.

## CORS

To enable CORS headers add the following section to your configuration.

```yaml
version: "3"

http:
  address: 127.0.0.1:44933
  middleware: ["headers"]
  # ...
  headers:
    cors:
      allowed_origin: "*"
      # If `allowed_origin_regex` option is set, the content of `allowed_origin` is ignored
      allowed_origin_regex: "^http://foo"
      allowed_headers: "*"
      allowed_methods: "GET,POST,PUT,DELETE"
      allow_credentials: true
      exposed_headers: "Cache-Control,Content-Language,Content-Type,Expires,Last-Modified,Pragma"
      max_age: 600
      # Status code to use for successful OPTIONS requests. Default value is 200.
      options_success_status: 200
      # Debugging flag adds additional output to debug server side CORS issues, consider disabling in production.
      debug: false
```

> Make sure to declare "headers" middleware.

> **Note**
> Since RoadRunner v2023.2.0 following changes were made:
> ability to define status code of successful OPTIONS request via options_success_status config;
> debug flag was added to enable additional output to debug CORS issues;
> it's allowed to define multiple allowed_origin values separated by comma;
> CORS requests are handled using [rs/cors](https://github.com/rs/cors) package.

## Custom headers for Response or Request

You can control additional headers to be set for outgoing responses and headers to be added to the request sent to your application.
```yaml
version: "3"

http:
  # ...
  headers:
      # Automatically add headers to every request passed to PHP.
      request:
        Example-Request-Header: "Value"
    
      # Automatically add headers to every response.
      response:
        X-Powered-By: "RoadRunner"
```
