## What is it?

RR plugin is a separate piece of software that can extend the functionality of RR. The plugin may depend on the other
plugins, or it may be completely independent.

### Creating own plugin/middleware:

- [Plugin](../customization/plugin.md)
- [Middleware](../customization/middleware.md)

### Interface

In general, every plugin implements the following set of methods. They are all optional, but add functionality to your plugin.
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