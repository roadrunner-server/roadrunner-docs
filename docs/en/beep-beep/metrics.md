# Application Metrics

RoadRunner server includes an embedded metrics server based on [Prometheus](https://prometheus.io/).

## Enable Metrics

To enable metrics add `metrics` section to your configuration:

```yaml
version: "2.7"

metrics:
  address: localhost:2112
```

Once complete you can access Prometheus metrics using `http://localhost:2112/metrics` url.

Make sure to install metrics extension:

```bash
composer require spiral/roadrunner-metrics
```

## Application metrics

You can also publish application-specific metrics using an RPC connection to the server. First, you have to register a metric in your
configuration file:

```yaml
version: "2.7"

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
version: "2.7"

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
