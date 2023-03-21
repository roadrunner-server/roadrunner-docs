# HTTP Middleware

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

> Middleware must correspond to the following [interface](https://github.com/roadrunner-server/api/blob/master/plugins/middleware/interface.go#L10) and be [named](https://github.com/roadrunner-server/endure/blob/master/pkg/container/container.go#L41).

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
		// Do something
}

// Middleware/plugin name.
func (p *Plugin) Name() string {
	return PluginName
}
```

> Middleware must correspond to the following [interface](https://github.com/roadrunner-server/grpc/blob/master/common/interfaces.go#L7) and be [named](https://github.com/roadrunner-server/endure/blob/master/pkg/container/container.go#L41).

---

You have to register this service after in the [container/plugin.go](https://github.com/roadrunner-server/roadrunner/blob/master/container/plugins.go) file in order to properly resolve dependency:

```golang
package roadrunner

import (
    "middleware"
)

func Plugins() []interface{} {
    return []interface{}{
    // ...
    
    // middleware
    &middleware.Plugin{},
    
    // ...
 }
```

Or you might use Velox to [build the RR binary](https://roadrunner.dev/docs/app-server-build/2.x/en).

You should also make sure you configure the middleware to be used via the [config or the command line](https://roadrunner.dev/docs/intro-config) otherwise the plugin will be loaded but the middleware will not be used with incoming requests.

```yaml
http:
    # provide the name of the plugin as provided by the plugin in the example's case, "middleware"
    middleware: [ "middleware" ]
```

### PSR7 Attributes

You can safely pass values to `ServerRequestInterface->getAttributes()` using [attributes](https://github.com/roadrunner-server/http/blob/master/attributes/attributes.go) package:

```golang
func (s *Service) Middleware(next http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		r = attributes.Init(r)
		attributes.Set(r, "key", "value")
		next.ServeHTTP(w, r)
	}
}
```