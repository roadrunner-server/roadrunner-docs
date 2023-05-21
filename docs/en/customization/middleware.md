# Customization — HTTP Middleware

RoadRunner provides a flexible and extensible architecture that allows developers to build custom middleware for
`http` and custom interceptors for `grpc` and `temporal` plugins. Moving highly loaded parts of an application, such as
authentication, to middleware written in Go can provide a significant performance boost. By leveraging the speed and
efficiency of Go, developers can improve the overall performance of their application and handle spikes in traffic more
effectively.

Middleware architecture allows developers to create custom middleware for their specific needs. The HTTP
middleware can be used to intercept and modify HTTP requests and responses, while the gRPC interceptor can be used to
intercept and modify gRPC requests and responses. This allows developers to add additional functionality to their
applications without having to modify the core application logic.

## HTTP

The HTTP middleware intercepts incoming HTTP requests and can be used to perform additional processing, such as
authentication, rate limiting, and logging.

**To create custom middleware for HTTP requests in RoadRunner, follow these steps:**

1. Define a struct that implements the `Init()`, `Middleware()`, and `Name()` methods. The `Init()` method is called
   when the plugin is initialized, the `Middleware()` method is called for each incoming HTTP request, and the `Name()`
   method returns the name of the middleware/plugin.

2. In the `Middleware()` method, perform any necessary processing on the incoming HTTP request, and then call the next
   middleware in the pipeline using the `next.ServeHTTP()` method.

**Here is an example:**

```go
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

> **Note**
> Middleware must correspond to the
> following [interface](https://github.com/roadrunner-server/http/blob/master/common/interfaces.go#L33) and
> be [named](https://github.com/roadrunner-server/endure/blob/master/container.go#L47).

## gRPC

The interceptor intercepts incoming gRPC requests and can be used to perform additional processing, such as
authentication, rate limiting, and logging.

**To create a custom interceptor for gRPC requests in RoadRunner, follow these steps:**

1. Define a struct that implements the `Init()`, `Interceptor()`, and `Name()` methods. The `Init() `method is called
   when the plugin is initialized, the `Interceptor()` method is called for each incoming gRPC request, and the `Name()`
   method returns the name of the middleware/plugin.

2. In the `Interceptor()` method, perform any necessary processing on the incoming gRPC request, and then call the next
   interceptor in the pipeline using the `handler(ctx, req)` method.

> **Warning**
> RoadRunner supports `gRPC` interceptors since `v2023.2.0` version.

**Here is an example:**

```go
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

> **Note**
> Interceptor must correspond to the
> following [interface](https://github.com/roadrunner-server/grpc/blob/master/common/interfaces.go#L14) and
> be [named](https://github.com/roadrunner-server/endure/blob/master/container.go#L47).

You can find a lot of examples here: [link](https://github.com/grpc-ecosystem/go-grpc-middleware). Keep in mind that, at
the moment, RR supports only `UnaryServerInterceptor` gRPC interceptors.

## PSR7 Attributes

PSR7 attributes are a way of attaching metadata to an incoming HTTP request or response. The PSR7 specification defines
a standard interface for HTTP messages, which includes the ability to set and retrieve attributes on both requests and
responses.

Attributes can be used to store any kind of metadata that might be useful for processing the request or response. For
example, you might use attributes to store information about the authenticated user, the user's IP address, or any other
custom data that you want to attach to the request.

The `Psr\Http\Message\ServerRequestInterface->getAttributes()` method can be used to retrieve attributes from an
incoming HTTP request, while the `ResponseInterface->withAttribute()` method can be used to set attributes on an
outgoing HTTP response.

You can safely pass values to a PHP application and retrieve attributes on PHP side using
the `Psr\Http\Message\ServerRequestInterface->getAttributes(`)
through [attributes](https://github.com/roadrunner-server/http/blob/master/attributes/attributes.go) package:

```go
func (s *Service) Middleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        r = attributes.Init(r)
        attributes.Set(r, "key", "value")
        next.ServeHTTP(w, r)
    }
}
```

> **Note**
> To retrieve the attributes in a PHP application, you would need to use a PSR-7 implementation that supports
> the `getAttributes()` method. For example, the `nyholm/psr7` package provides a PSR-7 implementation that supports it.

## Registering middleware

You have to register this service after in
the [container/plugin.go](https://github.com/roadrunner-server/roadrunner/blob/master/container/plugins.go) file in
order to properly resolve dependency:

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

Or you might use Velox tool to [build the RR binary](./build.md).

You should also make sure you configure the middleware to be used via
the [config or the command line](../intro/config.md). Otherwise, the plugin will be loaded, but the middleware will not
be used with incoming requests.

```yaml .rr.yaml
http:
  # provide the name of the plugin as provided by the plugin in the example's case, "middleware"
  middleware: [ "middleware" ]
```

## Video tutorial

### Writing a middleware for HTTP

[![Writing a middleware](https://img.youtube.com/vi/f5fUSYaDKxo/0.jpg)](https://www.youtube.com/watch?v=f5fUSYaDKxo)
