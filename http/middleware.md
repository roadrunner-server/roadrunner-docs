# HTTP Middleware
RoadRunner HTTP server uses default Golang middleware model which allows you to extend it using custom or 
community-driven middlewares. Simplest service with middleware registration would look like:

```golang
package middleware

import (
	"net/http"
)

const PluginName = "middleware"

type Plugin struct{}

// to declare plugin
func (g *Plugin) Init() error {
	return nil
}

func (g *Plugin) Middleware(next http.Handler) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		// do something
	}
}

// Middleware/plugin name.
func (g *Plugin) Name() string {
	return PluginName
}
```

> Middleware must correspond to the following [interface](https://github.com/spiral/roadrunner/blob/master/plugins/http/plugin.go#L44) and be named.

We have to register this service after in the `main.go` file in order to properly resolve dependency:

```golang
rr.Container.Register(http.ID, &http.Service{})
rr.Container.Register(custom.ID, &custom.Service{})

err = cli.Container.RegisterAll(		
		// ...
        &middleware.Plugin{},
	)
```

### PSR7 Attributes
You can safely pass values to `ServerRequestInterface->getAttributes()` using `attributes` package:

```golang
func (s *Service) middleware(f http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
	    attributes.Set(r, "key", "value")
	    f(w, r)
	}
}
```
