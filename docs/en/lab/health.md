# Observability — Health and Readiness checks

The RoadRunner Status Plugin provides a healthcheck status for various plugins such
as `http`, `grpc`, `temporal`, `jobs` and `centrifuge`. This plugin provides an easy way to check the condition of the
workers and ensure that they are ready to serve requests.

## Activation of the Status Plugin

To activate the health/readiness checks endpoint, include a `status` section in your configuration file.

Here is an example:

```yaml .rr.yaml
version: "3"

status:
  address: 127.0.0.1:2114
```

The above configuration sets the address to `127.0.0.1:2114`. This is the address that the plugin will listen to. You
can change the address to any IP address and port number of your choice.

To access the health check, you need to use the following URL: http://127.0.0.1:2114/health?plugin=http. This URL will
return the health status of the http plugin.

> **Note**
> You can specify multiple plugins by separating them with a comma. For example, to check the health status of both the
> http and grpc plugins, you can use the following URL: http://127.0.0.1:2114/health?plugin=http&plugin=grpc.

The health check endpoint will return `HTTP 200` if there is at least one worker ready to serve requests. If there are
no workers ready to service requests, the endpoint will return `HTTP 500`. If there are any other errors, the endpoint
will also return `HTTP 500`.

The readiness check endpoint will return `HTTP 200` if there is at least one worker ready to take the request (i.e., not
currently busy with another request). If there is no worker ready or all workers are busy, the endpoint will return 
`HTTP 500` status code (you can override this too).

## Customizing the Not-Ready Status Code

By default, Status Plugin uses a `500` status code. However, you can replace this status code with a custom one.

To achieve this, utilize the `unavailable_status_code` option:

```yaml .rr.yaml
version: "3"

status:
  address: 127.0.0.1:2114
  unavailable_status_code: 501
```

## Jobs plugin pipelines check

In addition to checking the health status of the workers, you can also examine the pipelines in the Jobs plugin using
the following URL: http://127.0.0.1:2114/jobs

This URL will return the status of the pipelines in the Jobs plugin. The output will be in the following format:

```
plugin: jobs: pipeline: test-1 | priority: 13 | ready: true | queue: test-1 | active: 0 | delayed: 0 | reserved: 0 | driver: memory | error:  
```

## Use cases

The health check endpoint serves the following purposes:

### Kubernetes Readiness and Liveness Probes

In Kubernetes, you can use readiness and liveness probes to check the health of your application. It can be used as a
readiness or liveness probe to ensure that your application is ready to serve requests. You can configure Kubernetes to
check the health check endpoint and take appropriate action if the endpoint returns an error.

**Read more [here](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)**

### AWS Target Group Health Checks

If you are using AWS Elastic Load Balancer, you can use it as a health check for your target group. You can configure
the target group to check the health check endpoint and take appropriate action if the endpoint returns an error.

**Read more [here](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html)**

### GCE Load Balancing Health Checks

If you are using Google Cloud Platform, you can use it as a health check for your load balancer. You can configure the
load balancer to check the health check endpoint and take appropriate action if the endpoint returns an error.

**Read more [here](https://cloud.google.com/load-balancing/docs/health-checks)**
