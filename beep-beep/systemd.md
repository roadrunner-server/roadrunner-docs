# Running a RR server as daemon on Linux

In the RR repository you can found rr.server systemd unit file. The structure of the file is the following:
```unit file (systemd)
[Unit]
Description=High-performance PHP application server

[Service]
Type=simple
ExecStart=/usr/local/bin/roadrunner serve -v -d -c <path/to/.rr.yaml>
Restart=always
RestartSec=30

[Install]
WantedBy=default.target 
```
The only thing that user should do is to update `ExecStart` option with you own. To do that, set a proper path of `roadrunner` binary, required flags and path to the .rr.yaml file.
Usually, such user owned unit files located at `.config/systemd/user/`, for RR it might be `.config/systemd/user/rr.service`. To enable it: `systemctl enable --user rr.service` and `systemctl start rr.service`. And that's it. Now roadrunner should run as daemon on your server.

Also, you can find more info here: [Link](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-managing_services_with_systemd-unit_files) and tune RR systemd unit file as fits you better.