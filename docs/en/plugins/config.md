# Plugins — Config

The config plugin is an essential component in RoadRunner, responsible for parsing configuration files and environment
variables. It serves as a central hub for managing configurations for the plugins and RoadRunner server itself.

## Configuration File Structure

The RoadRunner configuration file should be formatted using **YAML** or **JSON**. Each configuration file must include
a `version` at the top, indicating the format's version. The currently supported configuration version is **version 3**.

> **Warning**
> The configuration version is not synonymous with the RoadRunner (RR) version. The configuration version and RR version
> have separate versioning systems.

**Example of a YAML configuration file:**

```yaml
version: '3'

# ... other config values
```

> **Warning**
> Version numbers are strings, not numbers. For example, `version: "3"` is correct, but `version: 3` is not.

### Compatibility matrix

The compatibility matrix provides information about the supported configuration versions for different RoadRunner
versions.

| RR version     | Configuration version                                                          |
|----------------|--------------------------------------------------------------------------------|
| **>=2023.x.x** | **3**                                                                          |
| **>=2.8**      | **2.7**                                                                        |
| **2.7.x**      | **2.7** `OR` Unversioned (treated as `v2.6.0`, will be auto-updated to `v2.7`) |
| **<=2.6.x**    | Doesn't support versions                                                       |

> **Note**
> *non-versioned: configuration used in the 2.0.x-2.6.x releases.

## Changelog

### v3.0 Configuration

#### Reload plugin Update

⚠️ The `reload` plugin has been removed from the default plugins list. Please use `*.pool.debug=true` instead.

#### OpenTelemetry Middleware Update

Starting from version **v2023.1.0**, the OpenTelemetry (OTEL) middleware configuration has been moved out of the HTTP
plugin to support its usage across multiple plugins, including HTTP, gRPC, jobs and temporal. The OTEL middleware is now
configured using a top-level YAML key.

**RoadRunner 2.x**

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

**RoadRunner v2023.x.x**

```yaml
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

## Updating from `version: 2.7` to `version: 3`

To update your configuration from version 2.7 to version 3, follow these steps:

1. **Update the version number:** Change the `version` value from `2.7` to `3`.
2. **Relocate the `otel` middleware configuration:** If your configuration uses the `otel` middleware configuration
   within the `http` plugin, move it to the configuration root by cutting it from the `http` plugin and pasting it at
   the root level.
3. **Remove** the `reload` plugin configuration and if needed, use the `*.pool.debug=true` option instead.

## Tips

1. By default, `.rr.yaml` used as the configuration, located in the same directory with RR binary.

