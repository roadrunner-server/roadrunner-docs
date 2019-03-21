# HTTP Middleware
RoadRunner HTTP server uses default Golang middleware model which allows you to extend it using custom or community-driven middlewares. Simpliest service with middleware registration would look like:

```golang
package custom

import (
	rrttp "github.com/spiral/roadrunner/service/http"
	"net/http"
)

const ID = "custom"

type Service struct {
}

func (s *Service) Init(r *rrttp.Service) (bool, error) {
	r.AddMiddleware(s.middleware)
	return true, nil
}

func (s *Service) middleware(f http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
	    f(w, r)
	}
}
```

We have to register this service after `http` in the `main.go` file in order to properly resolve dependency:

```golang
rr.Container.Register(http.ID, &http.Service{})
rr.Container.Register(custom.ID, &custom.Service{})
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
