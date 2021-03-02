# Building a Server
RoadRunner use Endure to manage dependencies, this allows you to tweak and extend application functionality for each separate project.

#### Install Golang
To build an application server you need [Golang 1.16+](https://golang.org/dl/) to be installed.

#### Create main.go
Copy [main.go](https://github.com/spiral/roadrunner-binary/blob/main/main.go) file in the root of your project.

```golang
package main

import (
	"log"

	endure "github.com/spiral/endure/pkg/container"
	// plugins
	"github.com/spiral/roadrunner-binary/v2/cli"
	httpPlugin "github.com/spiral/roadrunner/v2/plugins/http"
	"github.com/spiral/roadrunner/v2/plugins/informer"
	"github.com/spiral/roadrunner/v2/plugins/kv/boltdb"
	"github.com/spiral/roadrunner/v2/plugins/kv/memcached"
	"github.com/spiral/roadrunner/v2/plugins/kv/memory"
	"github.com/spiral/roadrunner/v2/plugins/logger"
	"github.com/spiral/roadrunner/v2/plugins/metrics"
	"github.com/spiral/roadrunner/v2/plugins/redis"
	"github.com/spiral/roadrunner/v2/plugins/reload"
	"github.com/spiral/roadrunner/v2/plugins/resetter"
	"github.com/spiral/roadrunner/v2/plugins/rpc"
	"github.com/spiral/roadrunner/v2/plugins/server"
	"github.com/temporalio/roadrunner-temporal/activity"
	temporalClient "github.com/temporalio/roadrunner-temporal/client"
	"github.com/temporalio/roadrunner-temporal/workflow"
)

func main() {
	var err error
	cli.Container, err = endure.NewContainer(nil, endure.SetLogLevel(endure.ErrorLevel), endure.RetryOnFail(false))
	if err != nil {
		log.Fatal(err)
	}

	err = cli.Container.RegisterAll(
		// logger plugin
		&logger.ZapLogger{},
		// metrics plugin
		&metrics.Plugin{},
		// http server plugin
		&httpPlugin.Plugin{},
		// reload plugin
		&reload.Plugin{},
		// informer plugin (./rr workers, ./rr workers -i)
		&informer.Plugin{},
		// resetter plugin (./rr reset)
		&resetter.Plugin{},
		// rpc plugin (workers, reset)
		&rpc.Plugin{},
		// server plugin (NewWorker, NewWorkerPool)
		&server.Plugin{},

		// temporal plugins
		&temporalClient.Plugin{},
		&activity.Plugin{},
		&workflow.Plugin{},
	)
	if err != nil {
		log.Fatal(err)
	}

	cli.Execute()
}
```

You can now start your server without building `go run main.go serve`.

> See how to create [http middleware](/http/middleware.md) in order to intercept HTTP flow.