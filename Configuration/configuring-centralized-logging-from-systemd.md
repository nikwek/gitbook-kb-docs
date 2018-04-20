---
layout: default
category: configuration
title: Configuring centralized logging from systemd
sidebar_title: systemd
order: 165
---

# Configuring centralized logging from systemd

To send system logs from Linux distributions using systemd
(including newer releases of CentOS and CoreOS), follow the steps shown below
to forward output from `journalctl` to Papertrail.

For applications running in a Docker container, choose one of the options
described under the [Docker](/kb/configuration/configuring-centralized-logging-from-docker/) configuration page.

# Forwarding system logs

Create a new unit/service that pipes output from `journalctl` into `ncat`
or `socat` by following the steps below:

* Create a new unit file named `papertrail.service` under `/etc/systemd/system` (root permissions required).
* Paste the directives shown below, and replacing `logsN` and `XXXXX` with the values that appear under [log destinations](https://papertrailapp.com/account/destinations).

```
[Unit]
Description=Papertrail
After=systemd-journald.service
Requires=systemd-journald.service

[Service]
ExecStart=/bin/sh -c "journalctl -f | ncat --ssl logsN.papertrailapp.com XXXXX"
TimeoutStartSec=0
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

* Run `$ sudo systemctl enable /etc/systemd/system/papertrail.service` to enable the unit.
* Then run `$ sudo systemctl start papertrail.service` to start it.
* Finally, use `$ journalctl -f -u papertrail.service` to view the local logs and ensure the
new unit has started successfully

## Using `socat`

If `ncat` is not available, use `socat` instead by:

* saving [papertrail-bundle.pem](https://papertrailapp.com/tools/papertrail-bundle.pem) into `/etc/papertrail-bundle.pem`
* changing `ExecStart` to:

```
ExecStart=/bin/sh -c "journalctl -f | socat - SSL:logsN.papertrailapp.com:XXXXX,cafile=/etc/papertrail-bundle.pem"
```

Again, replace `logsN` and `XXXXX` with the values that appear under [log destinations](https://papertrailapp.com/account/destinations).