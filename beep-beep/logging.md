# Logging

RoadRunner provides the ability to control the log for each plugin individually.

## Global configuration

To configure logging globally use `logs` config section:

```yaml
logs:
  mode: production
  output: stderr
```

To use develop mode. It enables development mode (which makes DPanicLevel logs panic), uses a console encoder, writes to
standard error, and disables sampling. Stacktraces are automatically included on logs of WarnLevel and above.

```yaml
logs:
  mode: development
```

To output to separate file:

```yaml
logs:
  mode: production
  output: file.log
```

To use console friendly output:

```yaml
logs:
  encoding: console # default value
```

To suppress messages under specific log level:

```yaml
logs:
  encoding: console # default value
  level: info
```

## Channels

In addition, you can configure each plugin log messages individually using `channels` section:

```yaml
logs:
  encoding: console # default value
  level: info
  channels:
    server.mode: none # disable server logging. Also `off` can be used.
    http:
      mode: production
      output: http.log
```

## Summary:

1. Levels: `panic`, `error`, `warn`, `info`, `debug`. Default: `debug`.
2. Encodings: `console`, `json`. Default: `console`.
3. Modes: `production`, `development`. Default: `development`.
4. Output: `file.log` or `stderr`, `stdout`. Default `stderr`.
5. Error output: `err_file.log` or `stderr`, `stdout`. Default `stderr`.

> Feel free to register your own [ZapLogger](https://github.com/uber-go/zap) extensions.
