# Customization — Building a Server

Developers can take advantage of the customization options available with RoadRunner to create a server optimized
for their particular project.

**This can include:**

- adding custom plugins,
- forking existing ones to make changes,
- or building a lightweight server with only the necessary plugins.

We created a tool called **Velox** that lets developers build a RoadRunner server binary. It uses a configuration file
to determine which plugins and repositories are required for building a RoadRunner server binary.

## Configuration

The configuration file is written in TOML format and contains a list of repositories to add to the build. For each
repository, you can specify the owner and version. You can also add private repositories from Github or Gitlab, and
authenticate with access tokens.

> **Note**
> To download all the required plugins for RoadRunner, you need a GitHub token. If you try to download plugins without a
> token, anonymous access is limited to 50 requests per hour. You can read more about these limits on 
> the [Rate limits for GitHub Apps](https://docs.github.com/en/apps/creating-github-apps/setting-up-a-github-app/rate-limits-for-github-apps)
> page.


**Here is an example of a configuration file:**

```toml velox_rr_2023.toml
[velox]
build_args = [
    '-trimpath',
    '-ldflags',
    '-s -X github.com/roadrunner-server/roadrunner/v2023/internal/meta.version=${VERSION} -X github.com/roadrunner-server/roadrunner/v2023/internal/meta.buildTime=${TIME}'
]

[roadrunner]
ref = "v2023.1.3"

[github]
[github.token]
token = "${RT_TOKEN}"

[github.plugins]
# LOGS
appLogger = { ref = "v4.0.4", owner = "roadrunner-server", repository = "app-logger" }
logger = { ref = "v4.1.3", owner = "roadrunner-server", repository = "logger" }

# CENTRIFUGE BROADCASTING PLATFORM
centrifuge = { ref = "v4.1.2", owner = "roadrunner-server", repository = "centrifuge" }

# WORKFLOWS ENGINE
temporal = { ref = "v4.2.1", owner = "temporalio", repository = "roadrunner-temporal" }

# METRICS
metrics = { ref = "v4.0.4", owner = "roadrunner-server", repository = "metrics" }

# HTTP + MIDDLEWARE
http = { ref = "v4.1.4", owner = "roadrunner-server", repository = "http" }
gzip = { ref = "v4.0.4", owner = "roadrunner-server", repository = "gzip" }
prometheus = { ref = "v4.0.5", owner = "roadrunner-server", repository = "prometheus" }
headers = { ref = "v4.0.4", owner = "roadrunner-server", repository = "headers" }
static = { ref = "v4.0.5", owner = "roadrunner-server", repository = "static" }

# OpenTelemetry
otel = { ref = "v4.1.5", owner = "roadrunner-server", repository = "otel" }

# SERVER
server = { ref = "v4.1.1", owner = "roadrunner-server", repository = "server" }

# SERVICE aka lightweit systemd
service = { ref = "v4.1.0", owner = "roadrunner-server", repository = "service" }

# JOBS
jobs = { ref = "v4.3.2", owner = "roadrunner-server", repository = "jobs" }
amqp = { ref = "v4.4.3", owner = "roadrunner-server", repository = "amqp" }
sqs = { ref = "v4.2.3", owner = "roadrunner-server", repository = "sqs" }
beanstalk = { ref = "v4.2.2", owner = "roadrunner-server", repository = "beanstalk" }
nats = { ref = "v4.2.2", owner = "roadrunner-server", repository = "nats" }
kafka = { ref = "v4.1.4", owner = "roadrunner-server", repository = "kafka" }

# KV
kv = { ref = "v4.1.5", owner = "roadrunner-server", repository = "kv" }
boltdb = { ref = "v4.3.2", owner = "roadrunner-server", repository = "boltdb" }
memory = { ref = "v4.2.2", owner = "roadrunner-server", repository = "memory" }
redis = { ref = "v4.1.5", owner = "roadrunner-server", repository = "redis" }
memcached = { ref = "v4.1.5", owner = "roadrunner-server", repository = "memcached" }

# FILESERVER (static files)
fileserver = { ref = "v4.0.5", owner = "roadrunner-server", repository = "fileserver" }

# gRPC plugin
grpc = { ref = "v4.1.5", owner = "roadrunner-server", repository = "grpc" }

# HEALTHCHECKS + READINESS CHECKS
status = { ref = "v4.1.5", owner = "roadrunner-server", repository = "status" }

# TCP for the RAW TCP PAYLOADS
tcp = { ref = "v4.0.4", owner = "roadrunner-server", repository = "tcp" }

[gitlab]
[gitlab.token]
# api, read-api, read-repo
token = "${GL_TOKEN}"

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

> **Note**
> You can find the latest version of the example configuration file in
> the [official repository](https://github.com/roadrunner-server/velox/blob/master/velox_rr_v2023.toml).

> **Warning**
> When using official plugins for RoadRunner, it is recommended avoid using the `master` branch as it may contain
> unstable code. Instead, use tags with the same major version (e.g., `logger:v4.x.x` + `amqp:v4.x.x`, but
> not `logger:v4.0.0` + `amqp:v3.0.5`). Please note that the currently supported plugin version is `v4.x.x`, and the
> supported RoadRunner version is `v2023.x.x`.
>
> Failure to follow these guidelines may result in compatibility issues and
> other problems. Please pay close attention to your configuration file to ensure proper use of plugins.

You can use environment variables in the configuration file. This is useful when you want to keep the configuration file
in the repository, but you don't want to expose your tokens or just want to pass them as arguments to the `vx` command.

Here are the list of environment variables from the example above:

| Variable      | Description                                                                |
|---------------|----------------------------------------------------------------------------|
| `${GL_TOKEN}` | GitLab token.                                                              |
| `${RT_TOKEN}` | GitHub token.                                                              |
| `${VERSION}`  | RR version to write into the binary (will be shown with `./rr --version`). |
| `${TIME}`     | Build time (will be shown with `./rr --version`).                          |

> **Note**
> Keep in mind to set the latest stable version in the `${VERSION}` env variable. You may also use `${TIME}` env
> variable to write the build time in the output binary.

### Options:

| Option         | Description                                                                                                                                                                                                                  |
|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **ref**        | Tag, commit hash or branch name.                                                                                                                                                                                             |
| **owner**      | Repository owner (might be the user or organization).                                                                                                                                                                        |
| **repository** | Repository name.                                                                                                                                                                                                             |
| **folder**     | If the plugin is in some folder in your repository, you may specify it via this configuration option. <br/>For example: `cache = { ref = "v1.6.18", owner = "darkweak", repository = "souin", folder="plugins/roadrunner" }` |
| **replace**    | Go.mod [replace directive](https://go.dev/ref/mod#go-mod-file-replace).                                                                                                                                                      

### Private repositories

- Make sure the `ssh-agent` is running and the ssh key has been
  added: [link](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- Exclude your organization package prefix from the Go environment variables:

```
go env -w GOPRIVATE="gitlab/github.com/<company_name>/*"
go env -w GONOSUMDB="gitlab/github.com/<company_name>/*"
```

## Building

:::: tabs

::: tab Docker

Using the Docker image simplifies the build process by automatically building the RoadRunner binary and storing it in
the `/usr/bin/` folder. This eliminates the need to install Golang or other dependencies on your computer. Once the
build is complete, Docker will automatically start the RoadRunner server.

**Here is an example of Dockerfile**

```docker Dockerfile
# https://docs.docker.com/buildx/working-with-buildx/
# TARGETPLATFORM if not empty OR linux/amd64 by default
FROM --platform=${TARGETPLATFORM:-linux/amd64} ghcr.io/roadrunner-server/velox:latest as velox

# app version and build date must be passed during image building (version without any prefix).
# e.g.: `docker build --build-arg "APP_VERSION=1.2.3" --build-arg "BUILD_TIME=$(date +%FT%T%z)" .`
ARG APP_VERSION="undefined"
ARG BUILD_TIME="undefined"

# copy your configuration into the docker
COPY velox_rr_2023.toml .

# we don't need CGO
ENV CGO_ENABLED=0

# RUN build
RUN vx build -c velox_rr_2023.toml -o /usr/bin/

FROM --platform=${TARGETPLATFORM:-linux/amd64} php:8.2-cli

# copy required files from builder image
COPY --from=velox /usr/bin/rr /usr/bin/rr

# use roadrunner binary as image entrypoint
CMD ["/usr/bin/rr"]
```

:::

::: tab Go
You can use the `go install` command to download Velox.

> **Warning**
> To download Velox and build an application server you need [Golang 1.20+](https://golang.org/dl/) on you local server.

```shell
go install github.com/roadrunner-server/velox/cmd/vx@latest
```

After the binary has been downloaded, you can build the application server:

```bash
vx build -c velox_rr_2023.toml -o ~/Downloads
```

| Option | Description                     |
|--------|---------------------------------|
| `-c`   | path to the configuration       |
| `-o`   | path where to put the RR binary |

:::

::: tab Downloading binary
To build the application server, you need to download Velox binary
the [Github releases page](https://github.com/roadrunner-server/velox/releases) and unpack it to your `PATH`.

> **Warning**
> To build an application server you need [Golang 1.20+](https://golang.org/dl/) on you local server.

After the binary has been downloaded, you can build the application server:

```bash
vx build -c velox_rr_2023.toml -o ~/Downloads
```

| Option | Description                     |
|--------|---------------------------------|
| `-c`   | path to the configuration       |
| `-o`   | path where to put the RR binary |

:::

::::

## Video tutorials

### How to write a plugin

[![plugin](https://img.youtube.com/vi/h5PPvc_YOtg/0.jpg)](https://www.youtube.com/watch?v=h5PPvc_YOtg)

### `v2023.x.x` update

[![plugin](https://img.youtube.com/vi/w_uxFhdinvU/0.jpg)](https://www.youtube.com/watch?v=w_uxFhdinvU)

### Velox configuration

[![configuration](https://img.youtube.com/vi/sddi_lh7ePo/0.jpg)](https://www.youtube.com/watch?v=sddi_lh7ePo)

## Third party and deprecated plugins

- [souin, third party](https://github.com/darkweak/souin/tree/master/plugins/roadrunner)
- [reload, deprecated](https://github.com/roadrunner-server/reload)

## Known limitation

- At the moment only GitHub and GitLab repositories are supported.
