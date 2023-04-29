# RoadRunner — Configuration

RoadRunner supports both **YAML** and **JSON** configuration formats. The examples in our documentation use YAML, but
you can use JSON as well.

> **Note**
> To convert a YAML configuration file to JSON, you can use an online tool such
> as https://onlineyamltools.com/convert-yaml-to-json.

## Configuration reference

The most recent configuration reference with all available options can be found in the `.rr.yaml` file in the RoadRunner
GitHub repository:

- [**.rr.yaml**](https://github.com/roadrunner-server/roadrunner/blob/master/.rr.yaml)

> **Warning**
> We use dots as level separators, e.g.: `http.pool`, you can't use dots in section names, queue names,
> etc. You can find out more about it [here](https://github.com/roadrunner-server/roadrunner/issues/1529).

## Configuration file

RoadRunner looks for a configuration file named `.rr.yaml` in the current working directory. However, if you want to use
a configuration file with a different name or location, you can specify it using the `-c` option when starting the
server.

```terminal
./rr serve -c /path/to/file/.rr-dev.yaml
```

> **Note**
> Read more about starting the server in the [**Server Commands**](../app-server/cli.md) section.

## Environment variables

Environment variables allow you to separate configuration data from your application code, making it more maintainable
and portable.

RR supports the expansion of environment variables using the `${VARIABLE}` or `$VARIABLE` syntax. This feature can be
used in the `.rr.yaml` configuration file and CLI commands to dynamically set values based on the current environment.

### Configuration File

You can use environment variables in your `.rr.yaml` configuration file to configure various settings, such as the HTTP
address and port. This allows you to easily customize the configuration based on your specific environment without
changing the configuration file itself.

```yaml .rr.yaml
http:
  address: 127.0.0.1:${HTTP_PORT:-8080}
```

In this example, the `http.address` configuration will use port `8080` by default. However, you can redefine the port
using the `HTTP_PORT` environment variable.

Here's an example of a `docker-compose.yaml` file that redefines the `HTTP_PORT` for an RR service:

```yaml docker-compose.yaml
version: '3.8'

services:
  app:
    image: xxx
    environment:
      - HTTP_PORT=8081
...
```

### CLI Commands

You can also use environment variables in CLI commands to customize the behavior of your RR server. This is especially
useful when you need to pass configuration values that are environment-specific or sensitive, such as secrets or API
keys.

```bash
set -a
source /var/www/config/.env
set +a

exec /var/www/rr \
  -c /var/www/.rr.yaml \
  -w /var/www \
  -o http.pool.num_workers=${RR_NUM_WORKERS:-8} \
  -o http.pool.max_jobs=${RR_MAX_JOBS:-16} \
  -o http.pool.supervisor.max_worker_memory=${RR_MAX_WORKER_MEMORY:-512}
  serve
```

In this example, the following options are used:

| Option                   | Description                                                                                                                          |
|--------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| **-c**                   | Specifies the configuration file.                                                                                                    |
| **-w**                   | Specifies the working directory.                                                                                                     |
| **-o**                   | Overwrites specific configuration options.                                                                                           |
| **/var/www/config/.env** | File that contains the required environment variables.                                                                               |
| **${RR_NUM_WORKERS:-8}** | Sets the number of workers to `RR_NUM_WORKERS` from the `.env` file or uses the default value of `8` if the variable is not present. |

## What's Next?

1. [Server Commands](../app-server/cli.md) - learn how to start the server.
2. [Configuration plugin](../plugins/config.md) - learn more about the configuration plugin.
3. [PHP Workers — Environment configuration](../php/environment.md) - learn how to configure PHP workers environment.