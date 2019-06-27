# Application Metrics
RoadRunner server includes embedded metrics server based on [Prometheus](https://prometheus.io/).

## Enable Metrics
To enable metrics add `metrics` section to your configuration:

```yaml
metrics:
  address: localhost:2112
```

Once complete you can access Prometheus metrics using `http://localhost:2112/metrics` url. RoadRunner will expose default `http` and `limit` server metrics.

## Custom application metrics
You can also publish application specific metrics using RPC connection to the server. First you have to register metric in your
configuration file:

```yaml
metrics:
  address: localhost:2112
  collect:
    app_metric_counter:
      type: counter
      help: "Application counter."
```

To send metric from the application:

```php
$rpc = new Spiral\Goridge\RPC(new Spiral\Goridge\SocketRelay("127.0.0.1", 6001));

$metrics = new Spiral\RoadRunner\Metrics($rpc);
$metrics->add('app_metric_counter', 1);
```

> Supported types: gauge, counter, summary, histogram.

## Tagged metrics
You can use tagged (labels) metrics to group values:

```yaml
metrics:
  address: localhost:2112
  collect:
    app_type_duration:
      type: histogram
      help: "Application counter."
      labels: ["type"]
```

You should specify values for your labels while pushing the values:

```php
$rpc = new Spiral\Goridge\RPC(new Spiral\Goridge\SocketRelay("127.0.0.1", 6001));

$metrics = new Spiral\RoadRunner\Metrics($rpc);
$metrics->add('app_type_duration', 0.5, ['some-type']);
```
