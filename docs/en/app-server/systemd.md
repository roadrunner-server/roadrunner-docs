# App server — Running server as daemon on Linux

Here you can find an example of systemd unit file that can be used to run RoadRunner as a daemon on
a server:

```ini
[Unit]
Description = High-performance PHP application server

[Service]
Type = simple
ExecStart = /usr/local/bin/rr serve -c /var/www/.rr.yaml
Restart = always
RestartSec = 30

[Install]
WantedBy = default.target 
``` 

Where is

- `/usr/local/bin/rr` - path to the RoadRunner binary file
- `/var/www/.rr.yaml` - path to the RoadRunner configuration file

> **Warning**
> These paths are just examples, and the actual paths may differ depending on the specific
> server configuration and file locations. You should update these paths to match the actual paths used in your server
> setup.

You should also update the `ExecStart` option with your own configuration and save the file with a suitable name,
such as `rr.service`. Usually, such user unit files are located in the `.config/systemd/user/` directory. To enable the
service, you should run the following commands:

```bash
systemctl enable --user rr.service
```

and

```bash
systemctl start rr.service
``` 

This will start RoadRunner as a daemon on the server.

For more information about systemd unit files, the user can refer to the
following [link](https://wiki.archlinux.org/index.php/systemd#Writing_unit_files).

## SDNotify support

RR supports SDNotify protocol. You can use it to notify systemd about the readiness of your application. You don't need
to configure anything, RR will automatically detect systemd and send the notification. The only one option which might be
configured is watchdog timeout. By default, it's turned off. You can enable it by setting the following option in your 
`.rr.yaml` config:

```yaml
endure:
  log_level: error
  watchdog_sec: 60 # watchdog timeout in seconds
```