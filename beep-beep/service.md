# Service plugin

The service plugin was introduced in the RR `v2.5.0`.

### Main capabilities

1. Execute PHP code, binaries, bash/powershell scripts.
2. Restart after specified amount of time.
3. Control execute time for the particular command.
4. Provide statistic to the `Informer` plugin about `%CPU`, `PID` and used `RSS memory`.

### Config

```yaml
version: "2.7"

service:
  some_service_1:
    command: "php tests/plugins/service/test_files/loop.php"
    process_num: 10
    exec_timeout: 0
    remain_after_exit: true
    env:  
       - foo: "BAR"
    restart_sec: 1

  some_service_2:
    command: "tests/plugins/service/test_files/test_binary"
    process_num: 1
    remain_after_exit: true
    restart_delay: 1s
    env:
       - foo: "BAR"
    exec_timeout: 0
```

Description:

1. Service plugin supports any number of nested commands.
2. `command` - command to execute. There are no limitations on commands here. Here could be binary, PHP file, script,
   etc.
3. `process_num` - default: 1, number of processes for the command to fire.
4. `exec_timeout` - default: 0 (unlimited), maximum allowed time to run for the process.
5. `remain_after_exit` - default: false. Remain process after exit. For example, if you need to restart process every 10
   seconds
   `exec_timeout` should be 10s, and `remain_after_exit` should be set to true. NOTE: if you kill the process from
   outside and if `remain_after_exit` will be true, the process will be restarted.

6. `restart_sec` - default: 30 seconds. Delay between process stop and restart.
7. `env` - environment variables to pass to the underlying process from the config.
<!--
8. `working_dir` - default: null. A directory to which service should temporarily chdir before exec’ing the command.
-->

### RPC Interface

Since RR `v2.9` you have an ability to manage services via RPC interface.

All communication between PHP and GO made by the RPC calls with protobuf payloads. You can find versioned proto-payloads here: [Proto](https://github.com/roadrunner-server/api/blob/master/proto/service/v1/service.proto).

- `Create(in *serviceV1.Create, out *serviceV1.Response)  error` - Create a new service.
- `Restart(in *serviceV1.Service, out *serviceV1.Response) error` - Restart an exist service.
- `Terminate(in *serviceV1.Service, out *serviceV1.Response) error` - Terminate (Kill) an exist service.
- `Status(in *serviceV1.Service, out *serviceV1.Status) error` - Get status for an exist service.
- `List(_ *serviceV1.Service, out *serviceV1.List) error` - Get list of available services.


### Integration with PHP

If you need to manage RR services in your PHP project you can use composer package [`spiral/roadrunner-services`](https://packagist.org/packages/spiral/roadrunner-services) for that.


