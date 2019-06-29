# Running in Docker
RoadRunner can run in docker container.

Check pre-build docker repository: https://github.com/n1215/roadrunner-docker-skeleton

## Controller RoadRunner from outside
By default embedded RPC server will listen only localhost connections. In order to control RR from outside you must:

* Expose :6001 port from your container.
* Configure rr to listed on 0.0.0.0

```yaml
rpc:
  listen: tcp://:6001
```
