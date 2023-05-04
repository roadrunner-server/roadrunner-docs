# Logging — Application logger

The RoadRunner server has a useful `app-logger` plugin that allows users to send logs from their applications to the
RoadRunner server using an RPC interface. This plugin is enabled by default and does not require any additional
configurations. It can be used to observe all application and server logs in one place. This is especially useful when
debugging and monitoring applications.

> **Note**
> It will send raw messages to the RoadRunner `STDERR`

## Configuration

The `logs` section in the RoadRunner configuration file allows you to configure logging behavior for their application.

```yaml .rr.yaml
rpc:
  listen: tcp://127.0.0.1:6001

logs:
  channels:
    app:
      level: info
```

> **Warning**
> To interact with the RoadRunner app-logger plugin, you will need to have the RPC defined in the rpc configuration
> section. You can refer to the documentation page [here](../php/rpc.md) to learn more about the configuration.

The `level` key is used to specify the logging level for this channel. This means that only log messages with a severity
level of info or higher will be sent to this channel.

> **Note**
> Read more about logging in the [Logging — Logger](./logger.md) section.

## PHP client

The RoadRunner `app-logger` plugin comes with a convenient PHP package that simplifies the process of integrating the
plugin with your PHP application.

### Installation

To get started, you can install the package via Composer using the following command:

```terminal
composer require roadrunner-php/app-logger
```

### Usage

After the installation, you can create an instance of the `RoadRunner\Logger\Logger` class, which will allow you to use
the available class methods.

**Here is an example:**

```php
use Spiral\Goridge\RPC\RPC;
use RoadRunner\Logger\Logger;

$rpc = RPC::create('tcp://127.0.0.1:6001');

$logger = new Logger($rpc);

$logger->info('Hello, RoadRunner!');
$logger->warning('Something might be wrong...');
$logger->error('Houston, we have a problem!');
```

> **Note**
> You can refer to the documentation page [here](../php/rpc.md) to learn more about creating the RPC connection.

### Available methods

- `debug(string): void`: Sends a debug log message to the server
- `error(string): void`: Sends an error log message to the server
- `info(string): void`: Sends an info log message to the server
- `warning(string): void`: Sends a warning log message to the server
- `log(string): void`: Sends a log message directly to the `STDERR` of the server

## API

### RPC API

RoadRunner provides an RPC API, which allows you to manage app-logger in your applications using remote
procedure calls. The RPC API provides a set of methods that map to the available methods of
the `RoadRunner\Logger\Logger`class in PHP.

> **Note**
> All methods accept a `string` (which will be log message) as a first argument and a `bool` placeholder for the second
> arg.

#### Error

Method sends an `error` log message with the specified message to the RoadRunner server.

```go
func (r *RPC) Error(in string, _ *bool) error {}
```

#### Info

Method sends an `info` log message with the specified message to the RoadRunner server.

```go
func (r *RPC) Info(in string, _ *bool) error {}
```

#### Warning

Method sends a `warning` log message with the specified message to the RoadRunner server.

```go
func (r *RPC) Warning(in string, _ *bool) error {}
```

#### Debug

Method sends a `debug` log message with the specified message to the RoadRunner server.

```go
func (r *RPC) Debug(in string, _ *bool) error {}
```

#### Log

Method sends a log message with the specified message directly to the `STDERR` of the RoadRunner server.

```go
func (r *RPC) Log(in string, _ *bool) error {}
```