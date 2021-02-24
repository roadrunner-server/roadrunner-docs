# Logging
RoadRunner provides the ability to control the log for each plugin individually.

## Global configuration
To configure logging globally use `logs` config section:

```yaml
logs:
  mode: production
  output: stderr
```

To output to separate file:

```yaml
logs:
  mode: production
  output: file.log
```

To use console friendly outout:

```yaml
logs:
  mode: console # default value
```

To suppress messages under specific log level:

```yaml
logs:
  mode: console # default value
  level: info
```

## Channels
In addition, you can configure each plugin log messages individually using `channels` section:

```yaml
logs:
  mode: console # default value
  level: info
  channels:
    server.mode: none # disable server loggin
    http:
      mode: production
      output: http.log
```

> Feel free to register your own [ZapLogger](https://github.com/uber-go/zap) extensions.