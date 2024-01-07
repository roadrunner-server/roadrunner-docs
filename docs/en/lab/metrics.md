# Observability — Application Metrics

RoadRunner offers metrics plugin, which provides an embedded metrics server based
on [Prometheus](https://prometheus.io/). The metrics plugin allows users to monitor the performance and health of their
applications and servers by collecting and displaying various metrics.

## Observing Application Server Metrics

To observe application server metrics, you can enable the metrics plugin by adding a `metrics` section to your
configuration file.

**Here is an example:**

```yaml .rr.yaml
version: "3"

metrics:
  address: 127.0.0.1:2112
```

After enabling the metrics plugin, you can access the Prometheus metrics by visiting the http://127.0.0.1:2112/metrics
URL. The metrics plugin provides several general metrics that apply to all plugins and specific metrics for each
supported plugin.

### General Metrics

The general metrics provided by the metrics plugin include:

- `{{plugin}}_total_workers` - Total number of workers used by the plugin.
- `{{plugin}}_worker_memory_bytes` - Worker's current memory usage.
- `{{plugin}}_worker_state` - Worker's current state.
- `{{plugin}}_workers_invalid` - Number of workers currently in invalid, killing, destroyed, errored, inactive states.
- `{{plugin}}_workers_ready` - Number of workers currently in ready state.

> **Note**
> `{{plugin}}` is the concatenation of the rr + plugin name. For example, for the `jobs` plugin, the metric would
> be `rr_jobs_total_workers`.

### HTTP Metrics

> **Note**
> To enable specific HTTP metrics, you need to add an HTTP middleware called `http_metrics` to the `http.middleware`
> section of your configuration file.
> 
> **Here is an example:**
> ```yaml
> http:
>   middleware: [ "http_metrics" ]
> ```

The HTTP metrics provided by the metrics plugin include:

- `rr_http_requests_queue` - Total number of queued requests waiting for a worker.
- `rr_http_uptime_seconds` - Plugin uptime in seconds.
- `rr_http_no_free_workers_total` - Total number of NoFreeWorkers errors.
- `rr_http_request_total` - Total number of handled HTTP requests after server restart.
- `rr_http_request_duration_seconds` - HTTP request duration.

### gRPC Metrics

The gRPC metrics provided by the metrics plugin include:

- `rr_grpc_requests_queue` - Total number of queued requests waiting for a worker.
- `rr_gprc_request_total` - Total number of handled gRPC requests after server restart.
- `rr_grpc_request_duration_seconds` - gRPC request duration.

### JOBS Metrics

The JOBS metrics provided by the metrics plugin include:

- `rr_jobs_jobs_err` - Number of jobs that failed while processing in the worker.
- `rr_jobs_jobs_ok` - Number of successfully processed jobs.
- `rr_jobs_push_err` - Number of jobs that failed to push.
- `rr_jobs_push_latency` - Histogram that represents the latency for pushed operation. Available filters: driver, job (
  pipeline), source.
- `rr_jobs_push_latency_sum` - Histogram that represents the latency for pushed operation. Available filters: driver,
  job (pipeline), source.
- `rr_jobs_push_latency_count` - Histogram that represents the latency for pushed operation and the number of processed
  jobs for the metric. Available filters: driver, job (pipeline), source.

## Application Metrics

The RoadRunner metrics plugin also allows you to publish application metrics to the server via RPC and collect them in
Prometheus. To do this, you need to register collectors in your configuration file.

**Here is an example:**

```yaml .rr.yaml
version: "3"

metrics:
  address: 127.0.0.1:2112
  collect:
    registered_users:
      type: counter
      help: "Total registered users."
```

> **Note**
> Supported types for collectors include `gauge`, `counter`, `summary`, and `histogram`.

### Tagged metrics

You can also use tagged (labels) metrics to group values:

```yaml
version: "3"

metrics:
  address: 127.0.0.1:2112
  collect:
    registered_users:
      type: histogram
      help: "Total registered users."
      labels: [ "type", "is_admin" ]
```

In the example below we will show you how to send metrics into `registered_users` collector.

### PHP client

The RoadRunner metrics plugin comes with a convenient PHP package that simplifies the process of integrating the plugin
with your PHP application.

#### Installation

To get started, you can install the package via Composer using the following command:

```terminal
composer require spiral/roadrunner-metrics
```

#### Usage

After the installation, you can create an instance of the `Spiral\RoadRunner\Metrics\Metrics` class, which will allow
you to use the available class methods.

**Here is an example:**

```php
use Spiral\RoadRunner\Metrics\Metrics;
use Spiral\Goridge\RPC\RPC;

$metrics = new Metrics(RPC::create('127.0.0.1:6001'));

$metrics->add('registered_users', 1);
```

#### Labels

Using labeled metrics allows you to attach additional metadata to your metrics, which can be useful for filtering,
grouping, and aggregating the data.

**Some benefits of using labeled metrics include:**

- **Increased granularity:** You can attach multiple labels to a metric, allowing you to slice and dice the data in
  various ways.
- **Better organization:** Labels can help you group and organize your metrics, making it easier to find and understand
  the data you are looking for.
- **Simplified querying:** You can use labels to filter and aggregate your metric data, making it easier to extract
  meaningful insights from the data.

You can also specify labels for your metrics by passing an array of labels to the `add` method:

```php
// ...

$labels = ['guest', 'false'];

$metrics->add('registered_users', 1, $labels);
```

#### Declare metrics

In addition to sending metrics to the server, you can also declare metrics from your PHP application. This can be useful
if you want to dynamically declare custom metrics in your application.

**Here is an example of how to declare a custom metric:**

```php
use Spiral\RoadRunner\Metrics\Collector;

$metrics->declare(
    'earned_money',
    Collector::counter()->withHelp('Total earned money.'),
);

$metrics->add('earned_money', 100_000_000);
```

You can also declare labeled metrics:

```php
$metrics->declare(
    'registered_users',
    Collector::counter()->withHelp('Total registered users counter.')
        ->withLabels('type', 'is_admin'),
);
```

## API

### RPC API

RoadRunner provides an RPC API, which allows you to manage metrics in your applications using remote
procedure calls. The RPC API provides a set of methods that map to the available methods of
the `Spiral\RoadRunner\Metrics\Metrics` class in PHP.

#### Add

Method is used to add a new metric to the declared collector.

```go
func (r *rpc) Add(m *Metric, ok *bool) error {}
```

#### Sub

Method is used to subtract a value from a declared metric (for gauge metrics only).

```go
func (r *rpc) Sub(m *Metric, ok *bool) error {}
```

#### Set

Method is used to set the value of a metric (for gauge metrics only).

```go
func (r *rpc) Set(m *Metric, ok *bool) (err error) {}
```

#### Declare

Method is used to register a new collector in Prometheus.

```go
func (r *rpc) Declare(nc *NamedCollector, ok *bool) error {}
```

#### Unregister

Method is used to remove a collector from the Prometheus registry.

```go
func (r *rpc) Unregister(name string, ok *bool) error {}
```

#### Observe

Method is used to observe the value of a metric (for histogram and summary metrics only).

```go
func (r *rpc) Observe(m *Metric, ok *bool) error {}
```
