# RoadRunner — Configuration reference

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
etc. [link](https://github.com/roadrunner-server/roadrunner/issues/1529)

## Configuration file

RoadRunner looks for a configuration file named `.rr.yaml` in the current working directory. However, if you want to use
a configuration file with a different name or location, you can specify it using the `-c` option when starting the
server.

```terminal
./rr serve -c /path/to/file/.rr-dev.yaml
```

> **Note**
> Read more about starting the server in the [**Server Commands**](../app-server/cli.md) section.


## What's Next?

1. [Server Commands](../app-server/cli.md) - learn how to start the server.
2. [Configuration plugin](../plugins/config.md) - learn more about the configuration plugin.