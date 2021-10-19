### Access logs

RR starting from the `v2.5.0` will bring access logs support (turned off by default). They will include the following information:  

- `method` - http method.
- `remote_addr` - request remote address.
- `bytes_sent` - content-length,
- `http_host` - host.
- `request` - request Query.
- `time_local` - local time in Common Log Format.
- `request_length` - request body with headers size (content-len + size of all headers) in bytes. Max allowed headers size for the RR is 1MB.
- `request_time` - request processing time in seconds with a milliseconds' resolution.
- `status` - http response status.
- `http_user_agent` - http user [agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent)
- `http_referer` - http [referer](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer)

#### Configuration:

```yaml
http:
  address: 127.0.0.1:44933
  middleware: []
  access_logs: true 
  # ...
```