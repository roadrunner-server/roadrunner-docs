# HTTP Middleware

RoadRunner HTTP server uses default Golang middleware model which allows you to extend it using custom or
community-driven middleware. The simplest service with middleware registration would look like:

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

func (g *Plugin) Middleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    // do something
 })
}

// Middleware/plugin name.
func (g *Plugin) Name() string {
    return PluginName
}
```

> Middleware must correspond to the following [interface](https://github.com/spiral/roadrunner/blob/master/plugins/http/plugin.go#L37) and be named.

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
xif err != nil {
    panic(err)
}

err = container.RegisterAll(  
  // ...
        &middleware.Plugin{},
 )
```

### PSR7 Attributes

You can safely pass values to `ServerRequestInterface->getAttributes()` using [attributes](https://github.com/spiral/roadrunner/blob/master/plugins/http/attributes/attributes.go) package:

```golang
func (s *Service) middleware(next http.HandlerFunc) http.HandlerFunc {
 return func(w http.ResponseWriter, r *http.Request) {
     r = attributes.Init(r)
     attributes.Set(r, "key", "value")
            next(w, r)
 }
}
```
