# Logging — HTTP Access logs

RoadRunner has the HTTP access logs support, which provides detailed information about incoming HTTP requests and
responses.

> **Note**
> This feature is disabled by default, but it can be easily enabled by configuring the HTTP server.

## Enabling HTTP Access Logs

To enable HTTP access logs in RoadRunner, you need to modify the configuration file of the HTTP server.

**Here's an example configuration file:**

```yaml .rr.yaml
version: "3"

http:
  address: 127.0.0.1:8000
  access_logs: true
  # ...
```

Once enabled, RoadRunner will log the following information for each incoming HTTP request:

- `method` - HTTP method of the request
- `remote_addr` - Remote address of the client
- `bytes_sent` - Content length of the response
- `http_host` - Host name of the server
- `request` - Query string of the request
- `time_local` - Local time of the server in Common Log Format
- `request_length` - Request body size including headers in bytes (content-len + size of all headers) in bytes. Max allowed headers
  size for the RR is 1MB.
- `request_time` - Request processing time in seconds with a milliseconds' resolution
- `status` - HTTP response status
- `http_user_agent` - HTTP user agent [agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) string of the client
- `http_referer` - HTTP [referer](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer) string of the client
