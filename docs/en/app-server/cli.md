# Server Commands

RoadRunner application can be started by calling a simple command from the root of your PHP application.

```bash
$ rr serve 
```

You can also start RoadRunner using configuration from custom location:

```bash
$ rr serve -c ./app/.rr.yaml
```

Stopping RR:
- Send a `SIGINT` or `SIGTERM` syscalls to the main RoadRunner process. Inside a `k8s` for example, this is done automatically when you're stopping the pod.

- If you want to stop RR manually, you may hit a `ctrl+c` for the graceful stop or hit `ctrl+c` one more time to force stop.
- You may also use a commands:

```bash
$ rr serve -p
```
Where `-p` means: create a `.pid` file. And then:

```bash
$ rr stop
```
Or to force stop:

```bash
$ rr stop -f
```

To reload all RoadRunner plugins:

```bash
$ rr reset
```

To reload silently:

```bash
$ rr reset --silent
```

> You can attach this command as a file watcher in your IDE.

To reset only particular plugins:

```bash
$ rr reset http
```

To run golang pprof server (debug mode):

```bash
$ rr serve -d -c .rr.yaml
```

To view the status of all active workers in an interactive mode.

```bash
$ rr workers -i -c .rr.yaml
```

```bash
Workers of [http]:
+---------+-----------+---------+---------+-----------------+
|   PID   |  STATUS   |  EXECS  | MEMORY  |     CREATED     |
+---------+-----------+---------+---------+-----------------+
|    9440 | ready     |  42,320 | 31 MB   | 22 days ago     |
|    9447 | ready     |  42,329 | 31 MB   | 22 days ago     |
|    9454 | ready     |  42,306 | 31 MB   | 22 days ago     |
|    9461 | ready     |  42,316 | 31 MB   | 22 days ago     |
+---------+-----------+---------+---------+-----------------+
```

RoadRunner supports `.env` files. To read environment variables from the `.env` file, use the `--dotenv` CLI command:
```bash
rr serve --dotenv .env -c .rr.yaml
```

List of all commands with available options:
- `serve`:
  1. `-c`: path to the config file.
  2. `-w`: set the working directory.
  3. `--dotenv`: populate the process with env variables from the `.dotenv` file.
  4. `-d`: start a pprof server. Note, this is not `debug`, to use debug logs level, please, use logs: https://roadrunner.dev/docs/plugins-logger/2.x/en
  5. `-s`: silent mode.
  6. `-o`: to override configuration keys with your values, e.g. `-o=http.address=:8080` will override the `http.address` from the `.rr.yaml`. 
  7. `-p`: create a `.pid` file to use `./rr stop` later. 

- `reset`:
  1. `-c`: path to the config file.

- `workers`:
  1. `-c`: path to the config file.
  2. `-w`: set the working directory. 
  3. `-i`: interactive mode (update statistic every second).
- `stop`:
  1. `-f`: force stop.
  2. `-s`: silent mode.
