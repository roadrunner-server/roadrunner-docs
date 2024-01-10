# PHP Workers — Debugging

You can use RoadRunner scripts with xDebug extension. In order to enable configure your IDE to accept remote connections. 

> **Note:** 
> If you run multiple PHP processes you have to extend the maximum number of allowed connections to the number of 
> active workers, otherwise some calls would not be caught on your breakpoints.

![xdebug](https://user-images.githubusercontent.com/796136/46493729-c767b400-c819-11e8-9110-505a256994b0.png)

To activate xDebug make sure to set the `xdebug.mode=debug` in your `php.ini`. 

To enable xDebug in your application make sure to set ENV variable `XDEBUG_SESSION`:

```yaml .rr.yaml
rpc:
   listen: tcp://127.0.0.1:6001

server:
   command: "php worker.php"
   env:
     - XDEBUG_SESSION: 1

http:
   address: "0.0.0.0:8080"
   pool:
      num_workers: 1
      debug: true
```

Please, keep in mind this guide: [xdebug3](https://xdebug.org/docs/upgrade_guide).  

You should be able to use breakpoints and view state at this point.

## PhpStorm notes

```bash
export PHP_IDE_CONFIG="serverName=octane-app.test"
export XDEBUG_SESSION="mode=debug start_with_request=yes client_host=127.0.0.1 client_port=9003 idekey=PHPSTORM"

php -dvariables_order=EGPCS artisan octane:start --max-requests=250 --server=roadrunner --port=8000 --rpc-port=6001 --watch --workers=1
```

Feel free to check our community notes: [link](https://forum.spiral.dev/t/xdebug-integration/86/1)

## Jobs debugging

### Prerequisites

1. XDebug 3 installed and properly configured.
2. Number of jobs workers set to `1` with `jobs.pool.num_workers` configuration option in `.rr.yaml`.

### Debug process

If you have any active XDebug listener while starting RoadRunner with XDebug enabled — disable it. This will prevent
false-positive debug session.

Once RoadRunner starts all workers, enable XDebug listener and reset jobs workers with:

```terminal
./rr reset jobs
```

Now you should see debug session started:

1. Step over to some place where job task is being resolved with `$consumer->waitTask()`.
2. As soon as you reach it, debugger stops but session will still be active.
3. Trigger task consumption with either an HTTP request or a console command (depending on how your application works).
and continue to debug job worker as usual within active session started before.
4. You can continue to debug jobs as long as debug session active.

If connections session broken or timed out, you can repeat instruction above to reestablish connection by resetting jobs workers.
