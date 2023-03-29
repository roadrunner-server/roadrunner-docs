# Running a RR server as daemon on Linux

In the RR repository, you will find the `rr.server` systemd unit file. The structure of this file is as follows:

```ini
[Unit]
Description=High-performance PHP application server

[Service]
Type=simple
ExecStart=/usr/local/bin/roadrunner serve -c <path/to/.rr.yaml>
Restart=always
RestartSec=30

[Install]
WantedBy=default.target 
``` 

The only thing that user should do is to update `ExecStart` option with your own. To do that, set a proper path of `roadrunner` binary, required flags and path to the .rr.yaml file.
Usually, such user unit files are located in `.config/systemd/user/`. For RR, it might be `.config/systemd/user/rr.service`. To enable it use the following commands: `systemctl enable --user rr.service` and `systemctl start rr.service`. And that's it. Now roadrunner should run as daemon on your server.

Also, you can find more info about systemd unit files here: [Link](https://wiki.archlinux.org/index.php/systemd#Writing_unit_files).
