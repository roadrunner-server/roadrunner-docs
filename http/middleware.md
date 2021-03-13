# HTTP Middleware
RoadRunner HTTP server uses default Golang middleware model which allows you to extend it using custom or 
community-driven middlewares. The simplest service with middleware registration would look like:

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
container, err := endure.NewContainer(nil, endure.SetLogLevel(endure.ErrorLevel), endure.RetryOnFail(false))
if err != nil {
	panic(err)
}
err = container.Register(&http.Service{})
if err != nil {
    panic(err)
}
err = container.Register(&custom.Service{})
if err != nil {
    panic(err)
}

err = container.RegisterAll(		
		// ...
        &middleware.Plugin{},
	)
```

### PSR7 Attributes
You can safely pass values to `ServerRequestInterface->getAttributes()` using `attributes` package:

```golang
func (s *Service) middleware(next http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
	    attributes.Set(r, "key", "value")
            next(w, r)
	}
}
```
