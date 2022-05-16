# Building a Server

RoadRunner use Endure to manage dependencies, this allows you to tweak and extend application functionality for each separate project.

#### Install Golang

To build an application server you need [Golang 1.17+](https://golang.org/dl/) to be installed.

#### Step-by-step walkthrough  

1. Install the [velox](https://github.com/roadrunner-server/velox) `go install github.com/roadrunner-server/velox/vx@v1.0.0-beta.1`
2. Configure:

```toml
[velox]
build_args = ['-trimpath', '-ldflags', '-s -X github.com/roadrunner-server/roadrunner/v2/internal/meta.version=v2.8.0-alpha.1 -X github.com/roadrunner-server/roadrunner/v2/internal/meta.buildTime=today']

[roadrunner]
ref = "master"

[github_token]
token = ""

[log]
level = "debug"
mode = "development"

[plugins]
# ref -> master, commit or tag
logger = { ref = "master", owner = "roadrunner-server", repository = "logger" }
temporal = { ref = "master", owner = "temporalio", repository = "roadrunner-temporal" }
metrics = { ref = "master", owner = "roadrunner-server", repository = "metrics" }
cache = { ref = "master", owner = "roadrunner-server", repository = "cache" }
reload = { ref = "master", owner = "roadrunner-server", repository = "reload" }
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
```

3. Build RR:  `vx build -c plugins.toml -o ~/Downloads`  

Where:  
- -c - path to the configuration
- -o - path where to put the RR binary
