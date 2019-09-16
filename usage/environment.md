# Environment configuration
All RoadRunner workers will inherit the system configuration available for the parent server process. In addition, you can customize the set of env variables to be passed to your workers using part `env` of `.rr` configuration file.

```yaml
env:
   key: value
```

> All keys will be automatically uppercased!

### Additional Customizations
Internally, RoadRunner services receive configuration using `env.Environment` interface dependency. This allows you to create your own method to deliver env variables to workers:

```golang
type Environment interface {
	GetEnv() (map[string]string, error)
	SetEnv(key, value string)
}
```

```golang
func main() {
	rr.Logger.Formatter = &logrus.TextFormatter{ForceColors: true}

        // get env vars via consul
	rr.Container.Register(sd.ID, consul.NewService())

	rr.Container.Register(rpc.ID, &rpc.Service{})
	rr.Container.Register(http.ID, &http.Service{})
	rr.Container.Register(static.ID, &static.Service{})

	// you can register additional commands using cmd.CLI
	rr.Execute()
}
```
> You must remove default `env` component.

### Default ENV values
RoadRunner provides set of ENV values to help the PHP process to identify how to properly communicate with the server.

Key     | Description
---     | ---
RR      | Helps to identify that PHP process run under RR.
RR_HTTP | Helps to identify that PHP process expects to work under HTTP PSR7 server.
RR_RPC  | Contains RPC connection address when enabled.
