## What is it?

RR plugin is a separate piece of software that can extend the RR functionality. The plugin may depend on the other plugins, but also might be fully independent. 

## Plugin GitHub template

- [Repository](https://github.com/roadrunner-server/samples)

### Interface

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

// Init is mandatory to implement
type Plugin struct{}

func (p *Plugin) Init( /* deps here */) error {
	return nil
}
```

Structure name should be `Plugin` if you want to build RR with the [`velox`](https://github.com/roadrunner-server/velox) tool.  

The only required method for the plugin is `Init`. It can receive other plugins via its [API](https://github.com/roadrunner-server/api).
Users should not request a plugin directly; instead, they should use a repository with the plugin's API and request only the plugin's interface, not its implementation. 
For example, if the `logger` implementation is registered, any other plugin can request the logger via its `Init` function, like:

```go
package main

import (
	"go.uber.org/zap"
)

const pluginName string = "example"

type Plugin struct {
	log *zap.Logger
}

type Logger interface { // <-- Logger plugin implements this interface
	NamedLogger(name string) *zap.Logger
}

func (p *Plugin) Init(logger Logger) error { // <-- here we requested a logger interface implementation from the RR container
	p.log = logger.NamedLogger(pluginName)
	return nil
}
```

More about plugins can be found here: [link](https://github.com/roadrunner-server/endure/tree/master/examples)
