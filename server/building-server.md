# Building Server
RoadRunner use service bus model, this allows you to tweak and extend application functionality for each separate project.

You can achieve that by simply copying `main.go` file in the root of your project.

```golang
package main

import (
	"github.com/sirupsen/logrus"
	rr "github.com/spiral/roadrunner/cmd/rr/cmd"

	// services (plugins)
	"github.com/spiral/roadrunner/service/env"
	"github.com/spiral/roadrunner/service/http"
	"github.com/spiral/roadrunner/service/rpc"
	"github.com/spiral/roadrunner/service/static"

	// additional commands and debug handlers
	_ "github.com/spiral/roadrunner/cmd/rr/http"
)

func main() {
	rr.Container.Register(env.ID, &env.Service{})
	rr.Container.Register(rpc.ID, &rpc.Service{})
	rr.Container.Register(http.ID, &http.Service{})
	rr.Container.Register(static.ID, &static.Service{})

	rr.Logger.Formatter = &logrus.TextFormatter{ForceColors: true}

	// you can register additional commands using cmd.CLI
	rr.Execute()
}
```

You can now start your server without building `go run main.go serve -v -d`.

Use `rr.Container.Register` to add more services:

```golang
rr.Container.Register(custom.ID, &custom.Service{})
```

> See how to create [middlewares](middleware) in order to intercept HTTP flow.
