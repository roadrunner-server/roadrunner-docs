## OpenTelemetry middleware

OpenTelemetry middleware is a unified standard for the tracing, logging and metrics information. At the moment, only tracing information is stable and safe to use in production.


### PHP Library

PHP library in the alpha stage at the moment: [link](https://github.com/open-telemetry/opentelemetry-php).  
Thanks to [Brett McBride](https://github.com/brettmc), he created a rr-otel [PHP demo](https://github.com/brettmc/rr-otel-demo).

### Original issue

- https://github.com/roadrunner-server/roadrunner/issues/1027

### Configuration

OT is a middleware plugin which currently working with gRPC and HTTP plugins.

Example configuration for HTTP:
```yaml
version: "2.7"

rpc:
  listen: tcp://127.0.0.1:6001

server:
  command: "php otel_worker.php"
  env:
    - OTEL_SERVICE_NAME: php
    - OTEL_TRACES_EXPORTER: otlp
    - OTEL_EXPORTER_OTLP_PROTOCOL: http/protobuf
    - OTEL_EXPORTER_OTLP_ENDPOINT: http://127.0.0.1:4318
    - OTEL_PHP_TRACES_PROCESSOR: simple
  relay: pipes

http:
  address: 127.0.0.1:15389
  middleware: [gzip, otel]
  pool:
    num_workers: 10

otel:
  insecure: true
  compress: false
  exporter: otlp
  service_name: rr_test
  service_version: 1.0.0
  endpoint: 127.0.0.1:4318

logs:
  encoding: console
  level: debug
  mode: production
```

Example configuration for gRPC:
```yaml
version: "2.7"

rpc:
  listen: tcp://127.0.0.1:6001

server:
  command: "php otel_worker.php"
  relay: pipes

grpc:
  listen: "tcp://127.0.0.1:9001"
  proto: "service.proto"
  pool:
    num_workers: 10

otel:
  insecure: true
  compress: false
  exporter: otlp
  service_name: rr_test
  service_version: 1.0.0
  endpoint: 127.0.0.1:4317
  
logs:
  encoding: console
  level: debug
```

`otel` contains the following keys:
1. `insecure`: boolean, default `false`. Use insecure endpoints (http/https) or insecure gRPC.
2. `compress`: boolean, default `false`. Use gzip to compress the spans.
3. `exporter`: string, default `otlp`. Provides functionality to emit telemetry to consumers. Possible values: `otlp` (used for `new_relic`, `datadog`), `zipkin`, `stdout`, `jaeger` or `jaeger_agent` to use a Jaeger agent UDP endpoint.
4. `custom_url`: string, default empty. Used for the `http` client to override the default URL.
5. `client`: string, default `http`. Client to send the spans. Possible values: `http`, `grpc`.
6. `endpoint`: string, default `localhost:4318`. Consumer's endpoint.
7. `service_name`: string, default: `RoadRunner`. User's service name.
8. `service_version`: string, default `1.0.0`. User's service version.
9. `headers`: key-values map, default empty. User defined headers. new_relic `api-key` should be [here](https://docs.newrelic.com/docs/more-integrations/open-source-telemetry-integrations/opentelemetry/opentelemetry-setup/#review-settings). 

### OpenTelemetry environment variables

Env variables are used for the PHP process and can be passed via the `server` env variables.

```yaml
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
