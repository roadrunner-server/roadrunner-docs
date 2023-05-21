# Customization — Writing Plugins

RoadRunner provides the ability to create custom plugins, event listeners, middlewares, etc., that extend its
functionality. It uses Endure container to manage dependencies, this approach is similar to the PHP Container
implementation with automatic method injection.

**To create a custom plugin, you can follow these steps:**

- Define a struct with a public `Init` method that returns an error value.
- Implement the `Service` interface in your struct to provide the `Serve` and `Stop` methods.
- Request dependencies using their respective interfaces and inject them using Endure container.
- Register your plugin with RoadRunner by creating a custom version of the `main.go` file and [building it](build.md).

Below you can find more information about the plugin interface, how to define a plugin, and how to access other plugins.

## Interface

RoadRunner plugins are implemented using the Service interface, which provides the `Serve` and `Stop` methods for
starting and stopping the plugin. Additionally, plugins can implement other optional interfaces
like `Named`, `Provider`, `Weighted`, and `Collector`. These interfaces enable plugins to provide dependencies to other
plugins, define their weight in the plugin's topology, and collect plugins that implement specific interfaces.

**Here is an example:**

```go
package sample

import (
    "context"

    "github.com/roadrunner-server/endure/v2/dep"
)

type (
    // Service interface can be implemented by the plugin to use Start-Stop functionality
    Service interface {
        // Serve starts the plugin
        Serve() chan error
        // Stop stops the plugin
        Stop(context.Context) error
    }

    // Named -> Name of the service
    Named interface {
        // Name return user friendly name of the plugin
        Name() string
    }

    // Provider declares the ability to provide service edges of declared types.
    Provider interface {
        // Provides function return set of functions which provided dependencies to other plugins
        Provides() []*dep.Out
    }

    // Weighted is optional to implement, but when implemented the return value added during the topological sort
    Weighted interface {
        Weight() uint
    }

    // Collector declares the ability to accept the plugins which match the provided method signature.
    Collector interface {
        // Collects search for the plugins which implements given interfaces in the args
        Collects() []*dep.In
    }
)
```

## Plugin definition

To define a custom plugin, create a struct with a public `Init` method that returns an error value (you can
use `roadrunner-server/errors` as the `error` package). In this method, you can access other plugins by requesting
dependencies.

```go
package custom

const PluginName = "custom"

type Plugin struct{}

func (s *Plugin) Init() error {
    return nil
}
```

## Disabling plugin

Sometimes, you may want to disable a plugin at runtime based on certain conditions. For example, if there are no
configurations for the plugin, or if there is an initialization error, but you still don't want to stop the execution of
the server. In such cases, you can return the special type of error called `Disabled`, which can be found in
the `github.com/roadrunner-server/errors` package. This type of error can only be used in the `Init` function of the
plugin.

```go
package custom

import (
    "github.com/roadrunner-server/errors"
)

const PluginName = "custom"

type Configurer interface {
    // UnmarshalKey takes a single key and unmarshal it into a Struct.
    UnmarshalKey(name string, out any) error
    // Has checks if config section exists.
    Has(name string) bool
}

type Plugin struct{}

func (s *Plugin) Init(cfg Configurer) error {
    const op = errors.Op("custom_plugin_init")
    // In this sample code, we're checking with the help of the configurer plugin if the `custom` configuration section exists
    // and if not, disabling the plugin.
    if !cfg.Has(PluginName) {
        return errors.E(op, errors.Disabled)
    }

    return nil
}
```

## Dependencies

You can access other plugins by requesting dependencies in your `Init` method. All dependencies should be represented as
interfaces, and a plugin implementing this interface should be registered in the RR's container - Endure.

```go
package custom

import (
    "go.uber.org/zap"
)

type Configurer interface { // <-- config plugin implements
    // UnmarshalKey takes a single key and unmarshal it into a Struct.
    UnmarshalKey(name string, out any) error
    // Has checks if config section exists.
    Has(name string) bool
}

type Logger interface { // <-- logger plugin implements
    NamedLogger(name string) *zap.Logger
}

type Service struct{}

func (s *Service) Init(r Configurer, log Logger) error {
    return nil
}
```

## Configuration

In most cases, your services would require a set of configuration values. RoadRunner can automatically populate and 
validate your configuration structure using the `config` plugin via an interface.

### YAML configuration sample

```yaml
custom:
  address: tcp://127.0.0.1:8888
```

### Plugin

```go
package custom

// file: plugin.go

import (
    "go.uber.org/zap"
    "github.com/roadrunner-server/errors"
)

const PluginName = "custom"

type Configurer interface { // <-- config plugin implements
    // UnmarshalKey takes a single key and unmarshal it into a Struct.
    UnmarshalKey(name string, out any) error
    // Has checks if config section exists.
    Has(name string) bool
}

type Logger interface { // <-- logger plugin implements
    NamedLogger(name string) *zap.Logger
}

type Plugin struct {
    cfg *Config
}

// Init plugin
// file: plugin.go
func (s *Plugin) Init(cfg Configurer, log Logger) error {
    const op = errors.Op("custom_plugin_init") // error operation name
    if !cfg.Has(PluginName) {
        return errors.E(op, errors.Disabled)
    }

    // unmarshall initial configuration
    err := cfg.UnmarshalKey(PluginName, &s.cfg)
    if err != nil {
        // Error will stop execution
        return errors.E(op, err)
    }

    // Check the unmarshalled configuration and fill-up the defaults if not provided by the configuration
    s.cfg.InitDefaults()

    return nil
}
```

### Configuration

```go
package custom

// file: config.go

type Config struct {
    Address string `mapstructure:"address"`
}

// InitDefaults .. You can also initialize some defaults values for config keys
func (cfg *Config) InitDefaults() {
    if cfg.Address == "" {
        cfg.Address = "tcp://127.0.0.1:8088"
    }
}
```

## Serving

Create `Serve` and `Stop` methods in your structure to let RoadRunner start and stop your service. You may also use a
context from the `Stop` method to let RR force your plugin to stop after a specified timeout in the configuration.

```yaml
## RoadRunner internal container configuration (docs: https://github.com/spiral/endure).
endure:
  # How long to wait for stopping.
  #
  # Default: 30s
  grace_period: 30s
```

### Plugin

```go
package custom

import (
    "context"
)

type Plugin struct{}

func (s *Plugin) Serve() chan error {
    const op = errors.Op("custom_plugin_serve")
    errCh := make(chan error, 1)

    err := s.DoSomeWork()
    if err != nil {
        errCh <- errors.E(op, err)
        return errCh
    }

    return nil
}

func (s *Plugin) Stop(ctx context.Context) error {
    return s.stopServing()
}

func (s *Plugin) DoSomeWork() error {
    return nil
}
```

The `Serve` method is thread-safe. It runs in a separate goroutine managed by the `Endure` container.
One note is that you should unblock it when calling `Stop` on the container.
Otherwise, the service will be killed after the timeout (which can be set in Endure).

## Collecting dependencies in runtime

RoadRunner provides a way to collect dependencies at runtime via the `Collects` interface.
This is very useful for middlewares or extending plugins with additional functionality without changing them.

Let's create an HTTP middleware:

1. Declare a required interface

```go
package custom

import (
    "net/http"
)

// Middleware interface
type Middleware interface {
    Middleware(f http.Handler) http.HandlerFunc
}
```

2. Implement `Collects` endure interface in the plugin where you want to have these dependencies in the runtime.

```go
package custom

// Collects collecting http middlewares
func (p *Plugin) Collects() []*dep.In {
    return []*dep.In{
        dep.Fits(func(pp any) {
            mdw := pp.(Middleware)
            // add the middleware to the list of the middleware
        }, (*Middleware)(nil)),
    }
}
```

Important notes:

1. `dep.Fits`: method used to check all registered plugins that fit the specified interface.
2. `func(pp any){}`: is a callback. You can pass an existing method with a `func (_ any)` signature or anonymous as in
   the example.
3. `(*Middleware)(nil)`: is the second argument of the `dep.Fits` method which should be an interface you want to find
   in the registered plugins.

## RPC Methods

Extending your plugin with RPC methods does not change the plugin at all. The only thing you have to do is to create a
file with RPC methods (let's call it `rpc.go`) and add all RPC methods for the plugin without modifying the plugin 
itself.

**Example based on the `informer` plugin:**

Suppose we have created a file `rpc.go`. The next step is to create a structure:

1. Create a structure: (logger is optional)

```go
package custom

import (
    "go.uber.org/zap"
)

type rpc struct {
    plugin *Plugin
    log    *zap.Logger
}
```

2. Create a method, which you want to expose:

```go
package custom

func (s *rpc) Hello(input string, output *string) error {
    *output = input
    // s.plugin.Foo() <-- you may also use methods from the Plugin itself
    s.log.Debug("foo")
    return nil
}
```

3. Create a method called `RPC` that accepts nothing and returns `any`:

```go
package custom

func (p *Plugin) RPC() any {
    return &rpc{srv: p, log: p.log}
}
```

RPC plugin will automatically find and register
your [RPC](https://github.com/roadrunner-server/rpc/blob/master/plugin.go#L184) methods under your plugin name. So, for
example, to call the `Hello` method you might use the following sample:

```php
var_dump($rpc->call('custom.Hello', 'world'));
```

## Tips:

1. More about plugins can be found here: [link](https://github.com/roadrunner-server/endure/tree/master/examples)