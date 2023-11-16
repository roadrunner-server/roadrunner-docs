# Observability — OpenTelemetry

RoadRunner offers OTEL (OpenTelemetry) plugin, which provides a unified standard for tracing, logging, and metrics
information. This plugin allows you to send tracing data from RoadRunner to tracing collectors
like [New Relic](https://newrelic.com/), [Zipkin](https://zipkin.io), [Jaeger](https://www.jaegertracing.io/),
[Datadog](https://www.datadoghq.com/), and more.
Starting with `v2023.3`, the `Jaeger` exporter is deprecated. Please use `OTLP` instead: [docs](https://www.jaegertracing.io/docs/1.49/architecture/).


![OpenTelemetry](https://user-images.githubusercontent.com/773481/213914208-cd944ca8-f218-4baf-8a54-5a4e42a1ed40.jpg)

> **Note**
> Read more about OpenTelemetry on the [official site](https://opentelemetry.io/).

The OpenTelemetry plugin is designed to integrate with various tracing collectors to provide end-to-end tracing of
requests across multiple services. The plugin is built to support the OpenTelemetry standard, which provides a unified
way of collecting telemetry data across various languages and frameworks.

The plugin supports tracing, logging, and metrics data, but currently only tracing information is stable and safe to use
in production.

## Configuration

Here is an example configuration file:

```yaml .rr.yaml
version: "3"

otel:
  # https://github.com/open-telemetry/opentelemetry-specification/blob/v1.25.0/specification/resource/semantic_conventions/README.md
  resource:
    service_name: "rr_test"
    service_version: "1.0.0"
    service_namespace: "RR-Shop"
    service_instance_id: "UUID"
  insecure: true
  compress: false
  exporter: otlp
  endpoint: 127.0.0.1:4317
```

Once the plugin is activated, the `grpc` and `jobs` plugins will use the configuration to send tracing data to the
collector. The `http` plugin requires the `otel` middleware to be added to the middleware list.

**Here is an example:**

```yaml .rr.yaml
http:
  address: 127.0.0.1:15389
  middleware: [ gzip, otel ]

...
```

**The `otel` section of the configuration file contains the following options:**

| Option              | Description                                                                                                                                                                                         |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **insecure**        | a boolean that determines whether to use insecure endpoints (HTTP/HTTPS) or insecure gRPC. The default value is `false`.                                                                            |                                                                                                                                                                                                        |
| **compress**        | a boolean that determines whether to use gzip to compress the spans. The default value is `false`.                                                                                                  |
| **exporter**        | a string that provides functionality to emit telemetry to consumers. Possible values are `otlp` (used for New Relic, Datadog, Jaeger), `zipkin`, `stdout` or `stderr`. The default value is `otlp`. |
| **custom_url**      | a string that is used for the http client to override the default URL. The default value is `empty`.                                                                                                |
| **client**          | a string that determines the client to send the spans. Possible values are http and grpc. The default value is `http`.                                                                              |
| **endpoint**        | a string that specifies the consumer's endpoint. The default value is `127.0.0.1:4318`.                                                                                                             |
| **service_name**    | a string that specifies the user's service name. The default value is `RoadRunner`.                                                                                                                 |
| **service_version** | a string that specifies the user's service version. The default value is `1.0.0`.                                                                                                                   |
| **headers**         | a key-value map that contains user-defined headers. The `api-key` for New Relic should be here.                                                                                                     |
| **resource**        | a key-value map that contains OTEL resource (https://github.com/open-telemetry/opentelemetry-specification/blob/v1.25.0/specification/resource/semantic_conventions/README.md)                      |

## Collector

The OpenTelemetry Collector offers a vendor-agnostic implementation of how to receive, process and export telemetry
data. It removes the need to run, operate, and maintain multiple agents/collectors. This works with improved scalability
and supports open-source observability data formats (e.g. Jaeger, Prometheus, Fluent Bit, etc.) sending to one or more
open-source or commercial back-ends. The local Collector agent is the default location to which instrumentation
libraries export their telemetry data.

To start the collector, you can use the official Docker container `otel/opentelemetry-collector-contrib`.

**Here is an example `docker-compose.yaml`:**

```yaml docker-compose.yaml
version: "3.6"

services:
  collector:
    image: otel/opentelemetry-collector-contrib
    command: [ "--config=/etc/otel-collector-config.yml" ]
    volumes:
      - ./otel-collector-config.yml:/etc/otel-collector-config.yml
    ports:
      - "4318:4318"
      - "4317:4317"
```

> **Note**
> Read more about the OpenTelemetry Collector on the [official site](https://opentelemetry.io/docs/collector/).

### Collector Configuration

The collector is started with the `otel-collector-config.yml` configuration file, which specifies how the collector
should receive, process, and export the tracing data.

Here is an example configuration file that sends data to Zipkin and Datadog:

```yaml otel-collector-config.yml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  batch:
    timeout: 1s

exporters:
  logging:
    loglevel: debug

  zipkin:
    endpoint: "http://zipkin:9411/api/v2/spans"

  datadog:
    api:
      site: datadoghq.eu
      key: ...

  otlp:
    endpoint: https://otlp.eu01.nr-data.net:443
    headers:
      api-key: ...

service:
  pipelines:
    traces:
      receivers: [ otlp ]
      processors: [ batch ]
      exporters: [ zipkin, datadog, otlp, logging ]
```

| Option         | Description                                                                                                                                                                                                                                  |
|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **receivers**  | a key-value map that specifies the protocols the collector should use to receive the tracing data.                                                                                                                                           |
| **processors** | a key-value map that specifies the processors to apply to the tracing data before exporting it. In this example, it uses the batch processor, which batches tracing data before exporting it.                                                |
| **exporters**  | a key-value map that specifies the destination collectors to which the tracing data should be exported.  In this example, it exports the tracing data to `zipkin`, `datadog`, `otlp`, and `logging`.                                         |
| **service**    | a key-value map that specifies the service name and the pipelines through which the tracing data flows.  In this example, the traces pipeline includes the otlp receiver, batch processor, and zipkin, datadog, otlp, and logging exporters. |

> **Note**
> Read more about the OpenTelemetry Collector configuration on
> the [official site](https://opentelemetry.io/docs/collector/configuration/).

## PHP Client

The official [PHP SDK](https://github.com/open-telemetry/opentelemetry-php) for OpenTelemetry provides support for the
OpenTelemetry standard on the PHP side.

To configure your PHP application to use the RoadRunner OpenTelemetry plugin, you need to use environment variables.

**Here is an example of the required environment variables:**

```dotenv
# OpenTelemetry
OTEL_SERVICE_NAME=php-blog
OTEL_TRACES_EXPORTER=otlp
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
OTEL_EXPORTER_OTLP_ENDPOINT=http://127.0.0.1:4318 # Collector address
OTEL_PHP_TRACES_PROCESSOR=simple
```

You can pass these environment variables to your PHP application from the RoadRunner configuration using
the `server.env` option:

```yaml .rr.yaml
server:
  command: "php otel_worker.php"
  env:
    - OTEL_SERVICE_NAME: php
    - OTEL_TRACES_EXPORTER: otlp
    - OTEL_EXPORTER_OTLP_PROTOCOL: http/protobuf
    - OTEL_EXPORTER_OTLP_ENDPOINT: http://127.0.0.1:4318
    - OTEL_PHP_TRACES_PROCESSOR: simple
  relay: pipes
```

## Supported plugins:

Here is the list of currently supported plugin

| Plugin/Driver | Description                                                                                  |
|---------------|----------------------------------------------------------------------------------------------|
| **Redis**     | Redis driver, e.g.: [link](https://redis.uptrace.dev/guide/go-redis-monitoring.html#uptrace) |
| **Memcached** | Memcached driver. Native OTEL integration.                                                   |
| **In-Memory** | In-Memory KV/Jobs driver.                                                                    |
| **BoltDB**    | Jobs and KV drivers.                                                                         |
| **KV**        | KV PRC layer.                                                                                |
| **HTTP**      | HTTP plugin with all HTTP middleware (gzip, http_tracing, headers, etc).                     |
| **JOBS**      | JOBS plugin RPC layer.                                                                       |
| **AMQP**      | AMQP driver.                                                                                 |
| **SQS**       | SQS driver.                                                                                  |
| **Kafka**     | Kafka driver.                                                                                |
| **NATS**      | NATS driver.                                                                                 |
| **Beanstalk** | Beanstalk driver.                                                                            |




> **Note**
> Thanks to [Brett McBride](https://github.com/brettmc), he created a
> rr-otel [PHP demo](https://github.com/brettmc/rr-otel-demo).

## Original issue

- https://github.com/roadrunner-server/roadrunner/issues/1027
