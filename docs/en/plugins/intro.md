## What is it?

RR plugin is a separate piece of software that can extend the RR functionality. The plugin may depend on the other plugins, but also might be fully independent. 

### Interface

```go
package sample

type (
	// This is the main Endure service interface which may be implemented to Start (Serve) and Stop plugin (OPTIONAL)
	Service interface {
		// Serve
		Serve() chan error
		// Stop
		Stop() error
	}

	// Name of the service (OPTIONAL)
	Named interface {
		Name() string
	}

	// Provider declares the ability to provide dependencies to other plugins (OPTIONAL)
	Provider interface {
		Provides() []interface{}
	}

	// Collector declares the ability to accept the plugins which match the provided method signature (OPTIONAL)
	Collector interface {
		Collects() []interface{}
	}
)

// Init is mandatory to implement
type Plugin struct{}

func (p *Plugin) Init( /* deps here */) error {
	return nil
}
```

The only required method for the plugin is `Init`. It can receive other plugins via its [API](https://github.com/roadrunner-server/api). Users should not request a plugin directly, but use a repository with the plugin's API and request only the plugin's interface, not implementation. However, requesting implementation (structure pointer) is also possible. 

For example, if the `logger` implementation is registered, any other plugin can request the logger via its `Init` function, like:

```go
package main

type Plugin struct {
	
}

func (p *Plugin) Init(log *zap.Logger) error { // <-- here we requested a logger from the RR container
	return nil
}
```

More about plugins can be found here: [link](https://github.com/roadrunner-server/endure/tree/master/examples)