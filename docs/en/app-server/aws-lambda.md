# App server — AWS Lambda

RoadRunner can run PHP as AWS Lambda function.

### PHP Worker

PHP worker does not require any specific configuration to run inside a Lambda function. We can use the default snippet with
internal counter to demonstrate how workers are being reused:

```php
<?php
/**
 * @var Goridge\RelayInterface $relay
 */
use Spiral\Goridge;
use Spiral\RoadRunner;

ini_set('display_errors', 'stderr');
require __DIR__ . "/vendor/autoload.php";

$worker = RoadRunner\Worker::create();
$psr7 = new RoadRunner\Http\PSR7Worker(
    $worker,
    new \Nyholm\Psr7\Factory\Psr17Factory(),
    new \Nyholm\Psr7\Factory\Psr17Factory(),
    new \Nyholm\Psr7\Factory\Psr17Factory()
);

while ($req = $psr7->waitRequest()) {
    try {
        $resp = new \Nyholm\Psr7\Response();
        $resp->getBody()->write("hello world");

        $psr7->respond($resp);
    } catch (\Throwable $e) {
        $psr7->getWorker()->error((string)$e);
    }
}
```

Name this file `handler.php` and put it into the root of your project. Make sure to run:

```terminal
composer require spiral/roadrunner-http nyholm/psr7
```

### Application

We can create a simple application to demonstrate how it works:

1. You need three files, main.go with the `Endure` container:

```go
package main

import (
  _ "embed"
  "log"
  "log/slog"
  "os"
  "os/signal"
  "sync"
  "syscall"
  "time"

  "github.com/roadrunner-server/config/v4"
  "github.com/roadrunner-server/endure/v2"
  "github.com/roadrunner-server/logger/v4"
  "github.com/roadrunner-server/server/v4"
)

//go:embed .rr.yaml
var rrYaml []byte

func main() {
  _ = os.Setenv("PATH", os.Getenv("PATH")+":"+os.Getenv("LAMBDA_TASK_ROOT"))
  _ = os.Setenv("LD_LIBRARY_PATH", "./lib:/lib64:/usr/lib64")

  cont := endure.New(slog.LevelError)

  cfg := &config.Plugin{
    Version:   "2023.3.0",
    Timeout:   time.Second * 30,
    Prefix:    "rr",
    Type:      "yaml",
    ReadInCfg: rrYaml,
  }

  err := cont.RegisterAll(
    cfg,
    &logger.Plugin{},
    &Plugin{},
    &server.Plugin{},
  )
  if err != nil {
    log.Fatal(err)
  }

  err = cont.Init()
  if err != nil {
    log.Fatal(err)
  }

  ch, err := cont.Serve()
  if err != nil {
    log.Fatal(err)
  }

  sig := make(chan os.Signal, 1)
  signal.Notify(sig, os.Interrupt, syscall.SIGINT, syscall.SIGTERM)

  wg := &sync.WaitGroup{}
  wg.Add(1)

  go func() {
    defer wg.Done()
    for {
      select {
      case e := <-ch:
        err = cont.Stop()
        if err != nil {
          log.Println(e.Error.Error())
        }
      case <-sig:
        err = cont.Stop()
        if err != nil {
          log.Println(err.Error())
        }
        return
      }
    }
  }()

  wg.Wait()
}
```

2. And `Plugin` for the RR:

```go
package main

import (
  "context"
  "sync"
  "time"

  "github.com/goccy/go-json"
  "github.com/roadrunner-server/errors"
  "github.com/roadrunner-server/goridge/v3/pkg/frame"
  "github.com/roadrunner-server/sdk/v4/pool"
  "github.com/roadrunner-server/sdk/v4/worker"

  "github.com/aws/aws-lambda-go/events"
  "github.com/aws/aws-lambda-go/lambda"
  "github.com/roadrunner-server/sdk/v4/payload"
  poolImp "github.com/roadrunner-server/sdk/v4/pool/static_pool"
  "go.uber.org/zap"
)

const (
  pluginName string = "lambda"
)

type Plugin struct {
  mu      sync.Mutex
  log     *zap.Logger
  srv     Server
  pldPool sync.Pool
  wrkPool Pool
}

// Logger plugin
type Logger interface {
  NamedLogger(name string) *zap.Logger
}

type Pool interface {
  // Workers returns workers list associated with the pool.
  Workers() (workers []*worker.Process)
  // Exec payload
  Exec(ctx context.Context, p *payload.Payload, stopCh chan struct{}) (chan *poolImp.PExec, error)
  // RemoveWorker removes worker from the pool.
  RemoveWorker(ctx context.Context) error
  // AddWorker adds worker to the pool.
  AddWorker() error
  // Reset kill all workers inside the watcher and replaces with new
  Reset(ctx context.Context) error
  // Destroy all underlying stacks (but let them complete the task).
  Destroy(ctx context.Context)
}

// Server creates workers for the application.
type Server interface {
  NewPool(ctx context.Context, cfg *pool.Config, env map[string]string, _ *zap.Logger) (*poolImp.Pool, error)
}

func (p *Plugin) Init(srv Server, log Logger) error {
  p.srv = srv
  p.log = log.NamedLogger(pluginName)
  p.pldPool = sync.Pool{
    New: func() any {
      return &payload.Payload{
        Codec:   frame.CodecJSON,
        Context: make([]byte, 0, 100),
        Body:    make([]byte, 0, 100),
      }
    },
  }

  return nil
}

func (p *Plugin) Serve() chan error {
  errCh := make(chan error, 1)
  const op = errors.Op("plugin_serve")

  p.mu.Lock()
  defer p.mu.Unlock()

  var err error
  p.wrkPool, err = p.srv.NewPool(context.Background(), &pool.Config{
    NumWorkers:      4,
    AllocateTimeout: time.Second * 20,
    DestroyTimeout:  time.Second * 20,
  }, nil, nil)
  if err != nil {
    errCh <- errors.E(op, err)
    return errCh
  }

  go func() {
    // register handler
    lambda.Start(p.handler())
  }()

  return errCh
}

func (p *Plugin) Stop(ctx context.Context) error {
  p.mu.Lock()
  defer p.mu.Unlock()

  if p.wrkPool != nil {
    p.wrkPool.Destroy(ctx)
  }

  return nil
}

func (p *Plugin) handler() func(ctx context.Context, request events.APIGatewayV2HTTPRequest) (events.APIGatewayV2HTTPResponse, error) {
  return func(ctx context.Context, request events.APIGatewayV2HTTPRequest) (events.APIGatewayV2HTTPResponse, error) {
    requestJSON, err := json.Marshal(request)
    if err != nil {
      return events.APIGatewayV2HTTPResponse{Body: "", StatusCode: 500}, nil
    }

    ctxJSON, err := json.Marshal(ctx)
    if err != nil {
      return events.APIGatewayV2HTTPResponse{Body: "", StatusCode: 500}, nil
    }

    pld := p.getPld()
    defer p.putPld(pld)

    pld.Body = requestJSON
    pld.Context = ctxJSON

    re, err := p.wrkPool.Exec(ctx, pld, nil)
    if err != nil {
      return events.APIGatewayV2HTTPResponse{Body: "", StatusCode: 500}, nil
    }

    var r *payload.Payload

    select {
    case pl := <-re:
      if pl.Error() != nil {
        return events.APIGatewayV2HTTPResponse{Body: "", StatusCode: 500}, nil
      }
      // streaming is not supported
      if pl.Payload().Flags&frame.STREAM != 0 {
        return events.APIGatewayV2HTTPResponse{Body: "streaming is not supported", StatusCode: 500}, nil
      }

      // assign the payload
      r = pl.Payload()
    default:
      return events.APIGatewayV2HTTPResponse{Body: "worker empty response", StatusCode: 500}, nil
    }

    var response events.APIGatewayV2HTTPResponse
    err = json.Unmarshal(r.Body, &response)
    if err != nil {
      return events.APIGatewayV2HTTPResponse{Body: "", StatusCode: 500}, nil
    }
    return response, nil
  }
}

func (p *Plugin) putPld(pld *payload.Payload) {
  pld.Body = nil
  pld.Context = nil
  p.pldPool.Put(pld)
}

func (p *Plugin) getPld() *payload.Payload {
  pld := p.pldPool.Get().(*payload.Payload)
  return pld
}
```  

3. Config file, which can be embedded into the binary with [`embed`](https://pkg.go.dev/embed) import:

```yaml .rr.yaml
version: "3"

server:
  command: "php handler.php"
  relay: pipes
  relay_timeout: 60s

logs:
  mode: production
  level: error
  encoding: json
  output: stderr

endure:
  grace_period: 1s
```

Here you can use full advantage of the RoadRunner, you can include any plugin here and configure it with the embedded config (within reasonable limits).

To build and package your lambda function run:

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -ldflags "-s" -o bootstrap-amd64 main.go plugin.go
zip main.zip * -r
```

You can now upload and invoke your handler using simple string event.

## Repository with the full example

- [link](https://github.com/roadrunner-server/aws-lambda)

## Notes

There are multiple notes you have to acknowledge:

- Start with one worker per lambda function to control your memory usage.
- Make sure to include env variables listed in the code to properly resolve the location of PHP binary and its dependencies.
