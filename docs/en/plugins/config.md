The `config` plugin parses the configurations of other plugins and uses flags to locate the YAML config.

## Compatibility matrix

⚠️ Keep in mind that the `yaml` configuration version is not the same as the RR version. They have independent versions.

| RR version |                               Configuration version                           |
|-----------------------|--------------------------------------------------------------------|
|   **>=2023.x.x**   | **3**   |
|   **>=2.8**             |          **2.7**                                                |
|   **2.7.x**   |          **2.7** `OR` Unversioned (treated as `v2.6.0`, will be auto-updated to `v2.7`) |
|   **<=2.6.x**   |          Doesn't support versions |

*non-versioned: configuration used in the 2.0.x-2.6.x releases.

## v3.0 Configuration
#### CHANGES:

**OpenTelemetry Middleware Update:**

As of version `v2023.1.0`, the OpenTelemetry (OTEL) middleware has been separated from the HTTP plugin. It is now configured using a top-level YAML key.

- **`v2.x`**
```yaml
# HTTP plugin settings.
http:
  ...
  middleware: [ "otel" ]
  otel:
    insecure: true
    compress: false
    client: http
    exporter: otlp
    custom_url: ""
    service_name: "rr_test"
    service_version: "1.0.0"
    endpoint: "127.0.0.1:4318"
```

- **`v2023.x.x`**
```yaml
http:
  ...
  middleware: [ "xxx" ]

otel:
  insecure: true
  compress: false
  client: http
  exporter: otlp
  custom_url: ""
  service_name: "rr_test"
  service_version: "1.0.0"
  endpoint: "127.0.0.1:4318"
```


## Tips:

1. By default, `.rr.yaml` used as the configuration, located in the same directory with RR binary.

