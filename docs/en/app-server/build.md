# Building a Server

RoadRunner uses Endure to manage dependencies, this allows you to tweak and extend application functionality for each separate project.

## GitHub plugin template
- [Repository](https://github.com/roadrunner-server/plugin_template)

#### Install Golang

To build an application server you need [Golang 1.18+](https://golang.org/dl/) or Docker to be installed.

### 1. Installation:

- Docker:

```dockerfile
# https://docs.docker.com/buildx/working-with-buildx/
# TARGETPLATFORM if not empty OR linux/amd64 by default
FROM --platform=${TARGETPLATFORM:-linux/amd64} ghcr.io/roadrunner-server/velox:latest as velox

# app version and build date must be passed during image building (version without any prefix).
# e.g.: `docker build --build-arg "APP_VERSION=1.2.3" --build-arg "BUILD_TIME=$(date +%FT%T%z)" .`
ARG APP_VERSION="undefined"
ARG BUILD_TIME="undefined"

# copy your configuration into the docker
COPY velox.toml .

# we don't need CGO
ENV CGO_ENABLED=0

# RUN build
RUN vx build -c velox.toml -o /usr/bin/

FROM --platform=${TARGETPLATFORM:-linux/amd64} php:8.1-cli

# copy required files from builder image
COPY --from=velox /usr/bin/rr /usr/bin/rr

# use roadrunner binary as image entrypoint
CMD ["/usr/bin/rr"]
```

- `go install` command:
```shell
go install github.com/roadrunner-server/velox/vx@latest
```

- Or download velox binary from the [releases page](https://github.com/roadrunner-server/velox/releases) and unpack to your `PATH`.

## Configuration:

```toml
# filename - `plugins.toml`

[velox]
build_args = ['-trimpath', '-ldflags', '-s -X github.com/roadrunner-server/roadrunner/v2/internal/meta.version=v2.10.1 -X github.com/roadrunner-server/roadrunner/v2/internal/meta.buildTime=10:00:00']

[roadrunner]
ref = "v2.10.1"

[github]
    [github.token]
    token = "token"

    [github.plugins]
    # ref -> master, commit or tag
    logger = { ref = "master", owner = "roadrunner-server", repository = "logger" }
    temporal = { ref = "master", owner = "temporalio", repository = "roadrunner-temporal" }
    metrics = { ref = "master", owner = "roadrunner-server", repository = "metrics" }
    cache = { ref = "master", owner = "roadrunner-server", repository = "cache" }
    reload = { ref = "master", owner = "roadrunner-server", repository = "reload" }
    otel = { ref = "master", owner = "roadrunner-server", repository = "otel" }
    server = { ref = "master", owner = "roadrunner-server", repository = "server" }
    service = { ref = "master", owner = "roadrunner-server", repository = "service" }
    amqp = { ref = "master", owner = "roadrunner-server", repository = "amqp" }
    beanstalk = { ref = "master", owner = "roadrunner-server", repository = "beanstalk" }
    boltdb = { ref = "master", owner = "roadrunner-server", repository = "boltdb" }
    broadcast = { ref = "master", owner = "roadrunner-server", repository = "broadcast" }
    fileserver = { ref = "master", owner = "roadrunner-server", repository = "fileserver" }
    grpc = { ref = "master", owner = "roadrunner-server", repository = "grpc" }
    gzip = { ref = "master", owner = "roadrunner-server", repository = "gzip" }
    headers = { ref = "master", owner = "roadrunner-server", repository = "headers" }
    http = { ref = "master", owner = "roadrunner-server", repository = "http" }
    jobs = { ref = "master", owner = "roadrunner-server", repository = "jobs" }
    memory = { ref = "master", owner = "roadrunner-server", repository = "memory" }
    nats = { ref = "master", owner = "roadrunner-server", repository = "nats" }
    new_relic = { ref = "master", owner = "roadrunner-server", repository = "new_relic" }
    prometheus = { ref = "master", owner = "roadrunner-server", repository = "prometheus" }
    redis = { ref = "master", owner = "roadrunner-server", repository = "redis" }
    sqs = { ref = "master", owner = "roadrunner-server", repository = "sqs" }
    static = { ref = "master", owner = "roadrunner-server", repository = "static" }
    status = { ref = "master", owner = "roadrunner-server", repository = "status" }
    kv = { ref = "master", owner = "roadrunner-server", repository = "kv" }
    memcached = { ref = "master", owner = "roadrunner-server", repository = "memcached" }
    tcp = { ref = "master", owner = "roadrunner-server", repository = "tcp" }

[gitlab]
    [gitlab.token]
    # api, read-api, read-repo
    token = "token"

    [gitlab.endpoint]
    endpoint = "https://gitlab.com"

    [gitlab.plugins]
    # ref -> master, commit or tag
    test_plugin_1 = { ref = "main", owner = "rustatian", repository = "36405203" }
    test_plugin_2 = { ref = "main", owner = "rustatian", repository = "36405235" }

[log]
level = "debug"
mode = "development"
```

### 3. Usage:

```shell
vx build -c plugins.toml -o ~/Downloads
```
Where:
- `-c` - path to the configuration
- `-o` - path where to put the RR binary

### Known limitation
- At the moment only GitHub and GitLab repositories are supported.


# Embedding a Server

In some cases, it can be useful to embed a RoadRunner server inside another GO program. This is often the case in microservice architectures where you may have a mandated GO framework for all the apps. In such cases it might not be possible to run a stock roadrunner instance and the only choice is to run roadrunner inside the main app framework / program.

This is now possible in version 2.11 and newer. Let's pretend we have a GO app with an HTTP handler and we want to pass the request to PHP via RoadRunner running in the same go app.

```go
func handleRequest(w http.ResponseWriter, request *http.Request) {
    // Find a way to pass that to RoadRunner so PHP handles the request
}
```

## Create an RR instance

```go
overrides := []string{} // List of configuration overrides
plugins := roadrunner.DefaultPluginsList() // List of RR plugins to enable
rr, err := roadrunner.NewRR(".rr.yaml", overrides, plugins)
```

Here we use the default list of plugins. The same list of plugin you would get if you were to run `rr serve` with a stock roadrunner binary.

You can however chose only the plugins you want and add your own private plugins as well:

```go
overrides := []string{
    "http.address=127.0.0.1:4444", // example override to set the http address
    "http.pool.num_workers=4", // example override of how to set the number of php workers
} // List of configuration overrides
plugins := []interface{}{
    &informer.Plugin{},
    &resetter.Plugin{},
    // ...
    &httpPlugin.Plugin{},
    // ...
    &coolCompany.Plugin{},
}
rr, err := roadrunner.NewRR(".rr.yaml", overrides, plugins)
```

## Passing requests to RoadRunner

Roadrunner can respond to HTTP requests, but also gRPC ones or many more. Because this is all done via plugins that each listen to different types of requests, ports, etc...

So when we talk about passing a request to roadrunner, we're actually talking about passing the request to roadrunner's HTTP plugin. To do this, we need to keep a handle on the http plugin.

```go
overrides := []string{} // List of configuration overrides
httpPlugin := &httpPlugin.Plugin{},
plugins := []interface{}{
    &informer.Plugin{},
    &resetter.Plugin{},
    // ...
    httpPlugin,
    // ...
    &coolCompany.Plugin{},
}
rr, err := roadrunner.NewRR(".rr.yaml", overrides, plugins)
```

The HTTP plugin is itself an `http.Handler` so it's now very easy to use it to let roadrunner and PHP handle the request:


```go
overrides := []string{
    // override the http plugin's address value for the current run of the program
    "http.address=127.0.0.1:4444",
} // List of configuration overrides
httpPlugin := &httpPlugin.Plugin{},
plugins := []interface{}{
    &informer.Plugin{},
    &resetter.Plugin{},
    // ...
    httpPlugin,
    // ...
    &coolCompany.Plugin{},
}
rr, err := roadrunner.NewRR(".rr.yaml", overrides, plugins)
if err != nil {
    return err
}

func handleRequest(w http.ResponseWriter, request *http.Request) {
    return httpPlugin.ServeHTTP(w, request)
}
```

## Starting & Stopping Embedded Roadrunner

Once everything is ready, we can start the roadrunner instance:

```go
errCh := make(chan error, 1)
go func() {
    errCh <- rr.Serve()
}()
```

`rr.Serve()` will block until it returns an error or `nil` if it was stopped gracefully.

To gracefully stop the server, we simply call `rr.Stop()`

## Roadrunner State

When you run roadrunner, it goes through multiple phases of initialization, running, stopping etc...
Sometimes it is useful to know about those, be it for debugging, to know if you're ready to accept requests, or if you can gracefully shutdown the main program.

You can call `rr.CurrentState()` on your roadrunner instance to retrieve one of the following states:

```
package fsm
// github.com/roadrunner-server/endure/pkg/fsm

type State uint32

const (
	Uninitialized State = iota
	Initializing
	Initialized
	Starting
	Started
	Stopping
	Stopped
	Error
)
```
Additionally, the actual status name can be obtained via `rr.CurrentState().String()`.
