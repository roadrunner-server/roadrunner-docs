# Health Endpoint
RoadRunner server includes a health check endpoint that returns the health of the workers.

## Enable health

To enable the health check endpoint, add a `status` section to your configuration:

```yaml
version: "2.7"

status:
  address: localhost:2114
```

To access the health-check use the following URL:

`http://localhost:2114/health?plugin=http`

> You can check one or multiple plugins using health-check. Currently, only HTTP supported.

Once enabled, the health check endpoint will respond with the following: 

 - `HTTP 200` if there is at least **one worker** ready to serve requests.
 - `HTTP 500` if there are **no workers** ready to service requests.


To access the readiness-check use the following URL:

`http://localhost:2114/ready?plugin=http`


The difference between `ready` and `health` endpoints in the underlying checks.

For the `ready`, at least 1 worker should be in the `Ready` state (ready to accept a request). For the `health` check at least 1 worker should be in the `Active` state (serving the request).

From the user perspective, the `Ready` state means that the request might be sent and processed immediately, but the `Active` state means that the worker is working on the request and is healthy.
## Use cases

The health check endpoint can be used for the following:

 - [Kubernetes readiness and liveness probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
 - [AWS target group health checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html)
 - [GCE Load Balancing health checks](https://cloud.google.com/load-balancing/docs/health-checks)
