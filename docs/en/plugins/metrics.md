# Application Metrics

RoadRunner server includes an embedded metrics server based on [Prometheus](https://prometheus.io/).

## Enable Metrics

To enable metrics add `metrics` section to your configuration:

```yaml
version: "3"

metrics:
  address: 127.0.0.1:2112
```

Once complete, you can access Prometheus metrics using http://127.0.0.1:2112/metrics url. Make sure to install the metrics extension:

Make sure to install the metrics extension:

```bash
composer require spiral/roadrunner-metrics
```

## HTTP metrics

To enable HTTP metrics, add the `http_metrics` middleware as the left-most middleware.

```yaml
version: '3'
http:
  middleware: ["http_metrics"]

metrics:
  address: localhost:2112
```


## Application metrics

You can also publish application-specific metrics using an RPC connection to the server. First, you have to register a metric in your
configuration file:

```yaml
version: "3"

metrics:
  address: localhost:2112
  collect:
    app_metric_counter:
      type: counter
      help: "Application counter."
```

To send metric from the application:

```php
$metrics = new Spiral\RoadRunner\Metrics\Metrics(
    Spiral\Goridge\RPC\RPC::create(Spiral\RoadRunner\Environment::fromGlobals()->getRPCAddress())
);

$metrics->add('app_metric_counter', 1);
```

> Supported types: gauge, counter, summary, histogram.

## Tagged metrics

You can use tagged (labels) metrics to group values:

```yaml
version: "3"

metrics:
  address: localhost:2112
  collect:
    app_type_duration:
      type: histogram
      help: "Application counter."
      labels: ["label_1", "label_2"]
```

You should specify values for your labels while pushing the metric:

```php
$metrics = new Spiral\RoadRunner\Metrics\Metrics(
    Spiral\Goridge\RPC\RPC::create(Spiral\RoadRunner\Environment::fromGlobals()->getRPCAddress())
);
/**
 * @var array<array-key, string>
 */
$labels = ['label_1_value', 'label_2_value'];

$metrics->add('app_type_duration', 0.5, $labels);
```

## Declare metrics

You can declare metric from PHP application itself:

```php
$metrics->declare(
    'test',
    Spiral\RoadRunner\Metrics\Collector::counter()->withHelp('Test counter')
);
```

## Grafana dashboards and prometheus

### General metrics
1. `{{plugin}}_total_workers`: total number of workers used by the plugin.
2. `{{plugin}}_worker_memory_bytes`: worker current memory usage.
3. `{{plugin}}_worker_state`: worker current state.
4. `{{plugin}}_workers_invalid`: workers currently in invalid, killing, destroyed, errored, inactive states.
5. `{{plugin}}_workers_ready`: workers currently in ready state.

Where plugin is the concatenation of the `rr` + `plugin` name. For example: for the `jobs` plugin it would be `rr_jobs_total_workers`.

### HTTP Metrics
1. `rr_http_requests_queue`: total number of queued requests which are waiting for the worker.
2. `rr_http_uptime_seconds`: plugin uptime in seconds.
3. `rr_http_no_free_workers_total`: total number of NoFreeWorkers errors.
4. `rr_http_request_total`: total number of handled http requests after server restart.
5. `rr_http_request_duration_seconds`: HTTP request duration.

### gRPC Metrics
1. `rr_grpc_requests_queue`: total number of queued requests which are waiting for the worker.
2. `rr_gprc_request_total`: total number of handled http requests after server restart.
3. `rr_grpc_request_duration_seconds`: gRPC request duration.

### JOBS Metrics
1. `rr_jobs_jobs_err`: number of jobs error while processing in the worker.
2. `rr_jobs_jobs_ok`: number of successfully processed jobs.
3. `rr_jobs_push_err`: number of jobs push which was failed.
4. `rr_jobs_push_latency`: histogram represents latency for pushed operation. Available filters: `driver`, `job` (pipeline), `source`.
5. `rr_jobs_push_latency_sum`: histogram represents latency for pushed operation. Available filters: `driver`, `job` (pipeline), `source`.
6. `rr_jobs_push_latency_count`: histogram represents latency for pushed operation, number of the processed jobs for the metric. Available filters: `driver`, `job` (pipeline), `source`.