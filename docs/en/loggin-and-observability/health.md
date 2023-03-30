# Health and Readiness checks

The RoadRunner server features a health check endpoint, which provides information on the condition of the workers.

## Enable health/readiness checks

To activate the health/readiness checks endpoint, include a status section in your configuration:

```yaml
version: "3"

status:
  address: 127.0.0.1:2114
  unavailable_status_code: 500
```

To access the health check, utilize the following URL:

`http://127.0.0.1:2114/health?plugin=http`, where

- `/health`: is the endpoint for the healthcheck. You may also use `/ready` or `/jobs` (the jobs endpoint does not support a list of plugins).
- `?plugin=<plugin_name>`: list of the plugins with workers to check.

> **Note**
> You can use the health check to assess one or multiple plugins. Health/readiness checks can be applied to any plugin that has workers.

After activation, the health check endpoint will return the following status codes:

- `HTTP 200` if there is at least **one worker** ready to serve requests.
- `HTTP 500` if there are **no workers** ready to service requests.

## Jobs plugin pipelines check

Additionally, you can examine the pipelines in the `Jobs` plugin using the following URL:

- `http://127.0.0.1:2114/jobs`

> **Note**
> Output: `plugin: jobs: pipeline: test-1 | priority: 13 | ready: true | queue: test-1 | active: 0 | delayed: 0 | reserved: 0 | driver: memory | error:  `

## Custom not-ready status code

By default, RoadRunner employs a `500` status code for health/readiness checks if issues (no workers or error workers) arise with the workers. However, you can replace this status code with a custom one. To achieve this, utilize the `unavailable_status_code` option:

```yaml
version: "3"

status:
  address: 127.0.0.1:2114
  unavailable_status_code: 500 # <-- YOUR CUSTOM CODE
```


## Use cases

The health check endpoint serves the following purposes::

- [Kubernetes readiness and liveness probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [AWS target group health checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html)
- [GCE Load Balancing health checks](https://cloud.google.com/load-balancing/docs/health-checks)
