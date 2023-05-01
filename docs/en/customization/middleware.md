# Customization — HTTP Middleware

## Video tutorial

- #### Writing a middleware for HTTP
[![Writing a middleware](https://img.youtube.com/vi/f5fUSYaDKxo/0.jpg)](https://www.youtube.com/watch?v=f5fUSYaDKxo)  

The RoadRunner HTTP server employs the standard Golang middleware architecture, enabling seamless integration of custom or community-developed middleware. A basic example of a service that incorporates middleware registration is as follows:

### HTTP

```golang
package middleware

import (
	"net/http"
)

const PluginName = "middleware"

type Plugin struct{}

// to declare plugin
func (p *Plugin) Init() error {
	return nil
}

func (p *Plugin) Middleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// do something
		// ...
		// continue request through the middleware pipeline
		next.ServeHTTP(w, r)
	})
}

// Middleware/plugin name.
func (p *Plugin) Name() string {
	return PluginName
}
```

> **WARNING**
> While RoadRunner supports writing `gRPC` interceptors, you can use it only from version `v2023.2.0`.

> **INFO**
> Middleware must correspond to the following [interface](https://github.com/roadrunner-server/http/blob/master/common/interfaces.go#L33) and be [named](https://github.com/roadrunner-server/endure/blob/master/container.go#L47).

You can find a lot of examples here: [link](https://github.com/grpc-ecosystem/go-grpc-middleware). Keep in mind that, at the moment, RR supports only `UnaryServerInterceptor` gRPC interceptors.

### gRPC

```golang
package middleware

import (
	"net/http"
)

const PluginName = "interceptor"

type Plugin struct{}

// to declare plugin
func (p *Plugin) Init() error {
	return nil
}

func (p *Plugin) Interceptor() grpc.UnaryServerInterceptor {
		// Do something and return interceptor
}

// Middleware/plugin name.
func (p *Plugin) Name() string {
	return PluginName
}
```

> Middleware must correspond to the following [interface](https://github.com/roadrunner-server/grpc/blob/master/common/interfaces.go#L14) and be [named](https://github.com/roadrunner-server/endure/blob/master/container.go#L47).

---

You have to register this service after in the [container/plugin.go](https://github.com/roadrunner-server/roadrunner/blob/master/container/plugins.go) file in order to properly resolve dependency:

```go
package roadrunner

import (
    "middleware"
)

func Plugins() []any {
    return []any {
    // ...
    
    // middleware
    &middleware.Plugin{},
    
    // ...
 }
```

Or you might use Velox to [build the RR binary](build.md).

You should also make sure you configure the middleware to be used via the [config or the command line](../intro/config.md). Otherwise, the plugin will be loaded, but the middleware will not be used with incoming requests.

```yaml
http:
    # provide the name of the plugin as provided by the plugin in the example's case, "middleware"
    middleware: [ "middleware" ]
```

### PSR7 Attributes

You can safely pass values to `ServerRequestInterface->getAttributes()` using [attributes](https://github.com/roadrunner-server/http/blob/master/attributes/attributes.go) package:

```go
func (s *Service) Middleware(next http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		r = attributes.Init(r)
		attributes.Set(r, "key", "value")
		next.ServeHTTP(w, r)
	}
}
```
