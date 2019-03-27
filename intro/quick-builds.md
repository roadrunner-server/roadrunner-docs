# Quick Builds
You are able to build RoadRunner locally using simple composer command provided by the package.

```
$ vendor/bin/rr-build
```

RoadRunner binary file will appear in your working directory.

> Docker is required!

## Customizations
By default, quick builds will use only `http`, `static` services (default). You can create your own builds by placing `.build.json` file in the root of your project. For example to compile RoadRunner with GRPC support use:

```json
{
  "packages": [
    "github.com/spiral/roadrunner/service/env",
    "github.com/spiral/roadrunner/service/http",
    "github.com/spiral/roadrunner/service/rpc",
    "github.com/spiral/roadrunner/service/static",
    "github.com/spiral/php-grpc"
  ],
  "commands": [
    "github.com/spiral/roadrunner/cmd/rr/http",
    "github.com/spiral/php-grpc/cmd/rr-grpc/grpc"
  ],
  "register": [
    "rr.Container.Register(env.ID, &env.Service{})",
    "rr.Container.Register(rpc.ID, &rpc.Service{})",
    "rr.Container.Register(http.ID, &http.Service{})",
    "rr.Container.Register(static.ID, &static.Service{})",
    "rr.Container.Register(grpc.ID, &grpc.Service{})"
  ]
}
```

> Make sure to place `register` directives in proper order.

You can specify version of complied application using first argument and build config using second:

```
$ vendor/rr/rr-build with-grpc .build.json
```
