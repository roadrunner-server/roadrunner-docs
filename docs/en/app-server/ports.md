# Ports and Containers
By default, the embedded RPC server is configured to accept connections solely from the localhost. To enable remote control of the RR system from external sources, you must follow these steps:

* Expose 6001 port from your container.
* Configure rr to listen on 0.0.0.0

```yaml
version: "3"

rpc:
  listen: tcp://:6001
```

> Keep in mind that TCP communication generally exhibits slower performance compared to Unix sockets.
