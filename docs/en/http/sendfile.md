# HTTP — X-Sendfile middleware

The `Send` HTTP middleware and the `X-Sendfile` HTTP response headers are used to stream large files using the RoadRunner.
While the file is being streamed with the help of the RoadRunner, the PHP worker may be accepting the next request.

Original issue: [link](https://github.com/roadrunner-server/roadrunner-plugins/issues/9)
The middleware reads the file in 10MB chunks. For example, for the 5Gb file, only 10MB of RSS is used. If the file
is smaller than 10MB, the middleware adjusts the buffer to fit the file size.

## Similar approaches:

- [NGINX](https://www.nginx.com/resources/wiki/start/topics/examples/xsendfile/)
- [Apache2](https://tn123.org/mod_xsendfile/)

## Configuration

```yaml
version: "3"

http:
  address: 127.0.0.1:55555
  max_request_size: 1024
  access_logs: false
  middleware: [ "sendfile" ]

  pool:
    num_workers: 2
    max_jobs: 0
    allocate_timeout: 60s
    destroy_timeout: 60s
```


