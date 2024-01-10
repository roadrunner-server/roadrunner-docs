# Logging — Logger

Logger Plugin is responsible for collecting logs from server plugins and PHP application workers' `STDERR` and
displaying them in the RoadRunner `STDERR`/`STDOUT`. It comes with a variety of options that allow you to customize the
way your application logs are collected and displayed.

### Configuration

### Modes

```yaml
logs:
  mode: production
```

There are three available modes:

1. `production` - This mode uses logger settings that are optimized for production usage.
2. `development` - This mode is enabled by default and is designed for use during application development. In
   development mode, DPanicLevel logs panic, console colors are used, and logs are written to standard error. Sampling
   is disabled, and stack traces are automatically included on logs of WarnLevel and above.
3. `raw` - This mode displays messages as raw output without any formatting. This mode is useful in production
   environments where you need to parse logs programmatically.

> **Note**
> Use `production` mode in production environments. It is optimized for production usage.

### Encoding

Logger supports two types of encoding, `console` and `json`. By default, `console` encoding is used, which outputs logs
in a friendly format. JSON encoding, on the other hand, returns messages in a JSON Structured logging format. This
format presents log messages as JSON objects with key-value pairs representing each log message field, making them more
machine-readable and easier to process programmatically. JSON encoding is also better suited for production usage.

```yaml
logs:
  encoding: console
```

### Level

The level is used to specify the logging level. This means that only log messages with a severity level will be sent to
this channel. Available levels include `panic`, `error`, `warn`, `info`, and `debug`.

```yaml
logs:
  level: info
```

> **Note**
> The default level is `debug`.

### Output

By default, RoadRunner sends logs to `STDERR`. However, you can configure RoadRunner to send logs to `STDOUT` by using
the output key.

```yaml
logs:
  output: stdout
```

### Line Endings

It allows configuring custom line endings for the logger. By default, the plugin uses `\n` as the line ending. Note that the `\n` is a forced default. This means that if the value is empty, RoadRunner will still use `\n`. So no empty line endings are allowed.

```yaml
logs:
  line_ending: "\r\n"
```

### Channels

In addition, you can configure each plugin log messages individually using the `channels` section. It allows you to
customize the logger settings for each plugin independently. You can disable logging for a particular plugin or change
its log mode and output destination.

```yaml
version: "3"

logs:
  encoding: console # default value
  level: info
  mode: none # disable server logging. Also `off` can be used.
  channels:
    http:
      mode: production
      output: http.log
```

## File Logger

It is possible to redirect channels or the entire log output to a file. To use the file logger, you need to set
the `file_logger_options.log_output` option to the filename where you want to write the logs.

### Entire log

```yaml
logs:
  mode: development
  file_logger_options:
    log_output: "test.log"
    max_size: 10
    max_age: 24
    max_backups: 10
    compress: true
```

### Channel

You can also redirect a specific channel to a file. To do this, you need to specify the channel name in the `channels`

```yaml
logs:
  mode: development
  level: debug
  channels:
    http:
      file_logger_options:
        log_output: "test.log"
        max_size: 10
        max_age: 24
        max_backups: 10
        compress: true
```

### Available options

1. `log_output`: Filename is the file to write logs to in the same directory. It uses `processname-lumberjack.log` in
   `os.TempDir()` if empty.
2. `max_size`: is the maximum size in megabytes of the log file before it gets rotated. It defaults to 100 megabytes.
3. `max_age`: is the maximum number of days to retain old log files based on the timestamp encoded in their filename.
   Note that a day is defined as 24 hours and may not exactly correspond to calendar days due to daylight savings, leap
   seconds, etc. The default is not to remove old log files based on age.
4. `max_backups`: is the maximum number of old log files to retain. The default is to retain all old log files (though
   MaxAge may still cause them to get deleted.)
5. `compress`: determines if the rotated log files should be compressed using gzip. The default is not to perform
   compression.
6. `log_ending`: line ending to use in the logger. Default is new line - `\n`.

## ZapLogger

Feel free to register your own [ZapLogger](https://github.com/uber-go/zap) extensions.
