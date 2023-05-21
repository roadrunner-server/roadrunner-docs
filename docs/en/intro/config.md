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

The RoadRunner looks for a configuration file named `.rr.yaml` in the same directory as the server binary.

If your configuration file and other application files are located in a different directory than the binary, you can use
the `-w` option to specify the working directory.

```terminal
./rr serve -w /path/to/project
```

You can also use the `-c` option to specify the path to the configuration file if you don't want to specify the working
directory.

```terminal
./rr serve -c /path/to/project/.rr-dev.yaml
```

Or you can combine the `-c` and `-w` options to specify both the configuration file and the working directory:

```terminal
./rr serve -c .rr-dev.yaml -w /path/to/project
```

> **Note**
> Read more about starting the server in the [**Server Commands**](../app-server/cli.md) section.

## Environment variables

Environment variables allow you to separate configuration data from your application code, making it more maintainable
and portable.

RoadRunner supports the expansion of environment variables using the `${VARIABLE}` or `$VARIABLE` syntax in a
configuration file and CLI commands. You can use this feature to dynamically set values based on the current
environment, such as database connection strings, API keys, and other sensitive information.

You can specify a default value for an environment variable using the `${VARIABLE:-DEFAULT_VALUE}` syntax. For example,
if you want to use a default value of `8080` for the `HTTP_PORT` environment variable if it is not defined or is empty,
you can use the following configuration:

```yaml .rr.yaml
http:
  address: 127.0.0.1:${HTTP_PORT:-8080}
```

> **Note**
> You can find more information on Bash Environment Variable Defaults in
> the [Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html#Shell-Parameter-Expansion).

This allows you to easily customize the configuration based on your specific environment without changing the
configuration file itself.

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

### Dotenv

RoadRunner supports reading environment variables from `.env` files, which are typically used to store sensitive or
environment-specific variables outside your codebase.

To read environment variables from an `.env` file, you can use the `--dotenv` CLI option when starting RoadRunner.

```terminal
./rr serve --dotenv /var/www/config/.env
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

The `set -a` enables automatic exporting of variables. Any variables that are defined in `/var/www/config/.env` will be
automatically exported to the environment, making them available to any child processes that are executed from the
current shell. The final `set +a` command disables automatic exporting of variables, ensuring that only the variables
that were defined in `/var/www/config/.env` are exported, and preventing any unintended variables from leaking into the
environment.

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