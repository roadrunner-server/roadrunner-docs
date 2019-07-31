# Writing Services
RoadRunner uses a service bus to organize its internal services and their dependencies. This approach is similar to the PHP Container implementation with automatic method injection. You can create your own services, event listeners, middlewares, etc.

To define your service create a struct with public `Init` method with return values (`bool`, `error`):

```golang
package custom

const ID = "custom"

type Service struct{
}

func(s *Service) Init() (ok bool, err error) {
   return true, nil
}
```

Return `false` as your first argument in order to disable the service.

You can register your service by creating a custom version of `main.go` file and [building it](Building-Server).

### Dependencies
You can access other RoadRunner services by requesting dependencies in your `Init` method:

```golang
package custom

import (
    "github.com/spiral/roadrunner/service/rpc"
    rrhttp "github.com/spiral/roadrunner/service/http"
)

type Service struct{
}

func(s *Service) Init(r *rpc.Service, rr *rrhttp.Service) (ok bool, err error) {
   return true, nil
}
```

> Make sure to request dependency as pointer.


### Configuration
In most of the cases, your services would require a set of configuration values. RoadRunner can automatically populate and validate your configuration structure using `Hydrate` method:

```golang
package custom

import (
	"github.com/spiral/roadrunner/service"
)

type Config struct {
   Key string
}

func(c *Config) Hydrate(cfg service.Config) error {
	return cfg.Unmarshal(&c)
}
```

Add new section to `.rr` file under name of your service:

```yaml
custom:
    key: value
```

You can now request your config as simple dependency for your service:

```golang
func(s *Service) Init(r *rpc.Service, cfg *Config) (ok bool, err error) {
   fmt.Println(cfg.Key)
   return true, nil
}
```

### Serving
Create `Serve` and `Stop` method in your structure to let RoadRunner start and stop your service.

```golang
func(s *Service) Serve() error { 
   return s.serveSomething()
}

func(s *Service) Stop() { 
   s.stopServing()
}
```

### RPC Methods
You can expose set of RPC methods for your PHP workers by registering `net/rpc` handler using `rpc` service.

```golang
func(s *Service) Init(r *rpc.Service) (ok bool, err error) {
   r.Register("custom", &rpcService{})
   return true, nil
}
```

Where `rpcService` is:

```golang
package custom

type rpcService struct {
}

func (s *rpcService) Hello(input string, output *string) error {
    *output = input    
    return nil
}
```

To use it within PHP using `RPC` [instance](RPC-Integration):

```php
var_dump($rpc->call('custom.Hello', 'world'));
```
