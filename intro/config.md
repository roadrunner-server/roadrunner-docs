# Configuration

Each of RoadRunner plugins requires proper configuration. By default, such configuration is merged into one file which
must be located in the root of your project. Each service configuration is located under the designated section. The
config file must be named as `.rr.{format}` where the format is `yml`, `json` and others supported by `sp13f/viper`.

## Configuration Reference
This is full configuration reference which enables all RoadRunner features.

```yaml
rpc:
  listen: tcp://127.0.0.1:6001

server:
  command: "php tests/psr-worker-bench.php"
  # optional, only for linux under sudo user
  user: ""
  # optional
  group: ""
  env:
    - SOME_KEY: "SOME_VALUE"
    - SOME_KEY2: "SOME_VALUE2"
  relay: "pipes"
  relay_timeout: 20s

# optional for development
logs:
  # default
  mode: development
  level: debug
  encoding: console
  output: stderr
  err_output: stderr
  channels:
    http:
      mode: development
      level: panic
      encoding: console
      output: stdout
    server:
      mode: production
      level: info
      encoding: console
      output: stderr
    rpc:
      mode: production
      level: debug
      encoding: console
      output: stderr

# healthcheck
status:
  address: "0.0.0.0:8090"

# Workflow and activity mesh service
temporal:
  address: localhost:7233
  activities:
    # default - num of logical CPUs
    num_workers: 6
    # default 0 - no limit
    max_jobs: 0
    # default 1 minute
    allocate_timeout: 60s
    # default 1 minute
    destroy_timeout: 60s
    # supervisor used to control http workers
    supervisor:
      # watch_tick defines how often to check the state of the workers (seconds)
      watch_tick: 1s
      # ttl defines maximum time worker is allowed to live (seconds)
      ttl: 0
      # idle_ttl defines maximum duration worker can spend in idle mode after first use. Disabled when 0 (seconds)
      idle_ttl: 10s
      # exec_ttl defines maximum lifetime per job (seconds)
      exec_ttl: 10s
      # max_worker_memory limits memory usage per worker (MB)
      max_worker_memory: 100

http:
  # host and port separated by semicolon
  address: 127.0.0.1:44933
  max_request_size: 1024
  # middlewares for the http plugin, order matters
  middleware: [ "static", "gzip", "headers" ]
  # uploads
  uploads:
    forbid: [ ".php", ".exe", ".bat" ]  
  trusted_subnets:
    [
        "10.0.0.0/8",
        "127.0.0.0/8",
        "172.16.0.0/12",
        "192.168.0.0/16",
        "::1/128",
        "fc00::/7",
        "fe80::/10",
    ]
  # headers (middleware)
  headers:
    cors:
      allowed_origin: "*"
      allowed_headers: "*"
      allowed_methods: "GET,POST,PUT,DELETE"
      allow_credentials: true
      exposed_headers: "Cache-Control,Content-Language,Content-Type,Expires,Last-Modified,Pragma"
      max_age: 600
    request:
      input: "custom-header"
    response:
      output: "output-header"
  # http static (middleware)
  static:
    dir: "tests"
    forbid: [ ] # file extensions to forbid
    request:
      input: "custom-header"
    response:
      output: "output-header"

  pool:
    # default - num of logical CPUs
    num_workers: 6
    # default 0 - no limit
    max_jobs: 0
    # default 1 minute
    allocate_timeout: 60s
    # default 1 minute
    destroy_timeout: 60s
    # supervisor used to control http workers
    supervisor:
      # watch_tick defines how often to check the state of the workers (seconds)
      watch_tick: 1s
      # ttl defines maximum time worker is allowed to live (seconds)
      ttl: 0
      # idle_ttl defines maximum duration worker can spend in idle mode after first use. Disabled when 0 (seconds)
      idle_ttl: 10s
      # exec_ttl defines maximum lifetime per job (seconds)
      exec_ttl: 10s
      # max_worker_memory limits memory usage per worker (MB)
      max_worker_memory: 100

  ssl:
    # host and port separated by semicolon (default :443)
    address: :8892
    redirect: false
    cert: fixtures/server.crt
    key: fixtures/server.key
    root_ca: root.crt
    
  fcgi:
    address: tcp://0.0.0.0:7921
    
  http2:
    h2c: false
    max_concurrent_streams: 128

metrics:
  # prometheus client address (path /metrics added automatically)
  address: localhost:2112
  collect:
    app_metric:
      type: histogram
      help: "Custom application metric"
      labels: [ "type" ]
      buckets: [ 0.1, 0.2, 0.3, 1.0 ]
      # objectives defines the quantile rank estimates with their respective
      #	absolute error [ for summary only ]
      objectives:
        - 1.4: 2.3
        - 2.0: 1.4

reload:
  # sync interval
  interval: 1s
  # global patterns to sync
  patterns: [ ".php" ]
  # list of included for sync services
  plugins:
    http:
      # recursive search for file patterns to add
      recursive: true
      # ignored folders
      ignore: [ "vendor" ]
      # service specific file pattens to sync
      patterns: [ ".php", ".go", ".md" ]
      # directories to sync. If recursive is set to true,
      # recursive sync will be applied only to the directories in `dirs` section
      dirs: [ "." ]
```

## Minimal configuration
You are not required to enable every plugin to make RoadRunner work. Given configuration only enables essential plugins
to make HTTP endpoints work:

```yaml
rpc:
  listen: tcp://127.0.0.1:6001

server:
  command: "php tests/psr-worker-bench.php"

http:
  address: "0.0.0.0:8080"
  pool:
    num_workers: 4   
```

## Console flags

You can overwrite any of the config values using `-o` flag:

```
rr serve -o http.address=:80 -o http.workers.pool.numWorkers=1
```

> The values will be merged with `.rr` file.

## Environment Variables

RoadRunner will replace some config options using reference(s) to environment variable(s):

```yaml
http:
  pool.num_workers: ${NUM_WORKERS}
```
