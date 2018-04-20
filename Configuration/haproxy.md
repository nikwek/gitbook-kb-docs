---
layout: default
category: configuration
title: haproxy
order: 89
---

# haproxy

Aggregate logs from [haproxy](http://haproxy.1wt.eu/), a "reliable, high Performance TCP/HTTP load balancer," including HTTP requests and errors.

# Direct logging

`haproxy` supports remote syslog natively, so it can log directly to Papertrail. Edit `haproxy.cfg` and add a new global `log` configuration option like this, as well as `log-send-hostname`:

```
log logsN.papertrailapp.com:XXXXX local0
log-send-hostname
```

where `logsN` and `XXXXX` are the name and port number shown under [log destinations](https://papertrailapp.com/account/destinations).

Second, add `log global` to each server's configuration stanza to indicate that it should use the aforementioned configuration.

haproxy supports up to 2 concurrent `log` configuration outputs.

Read more: [log](https://cbonte.github.io/haproxy-dconv/1.7/configuration.html#8) configuration option

## Define syslog tag and/or hostname (optional)

Syslog messages include a program name ("tag") and sender identifier ("hostname"). With the configuration above, haproxy will use default values of `haproxy` and the system hostname. These are typically fine.

Changing the tag is typically only necessary if you run multiple haproxy instances on the same system. Use different tag values to tell their logs apart.

Changing the hostname is rarely needed, since the system hostname is usually the best identification.

To change them, use the [log-tag](https://cbonte.github.io/haproxy-dconv/1.7/configuration.html#3.1-log-tag) or [log-send-hostname](https://cbonte.github.io/haproxy-dconv/1.7/configuration.html#3.1-log-send-hostname) options.

## Change logging verbosity (optional)

Optionally append a minimum severity from among the [set](http://cbonte.github.com/haproxy-dconv/configuration-1.5.html#3-log):

```
log logsN.papertrailapp.com:XXXXX local0 warning
```

where `logsN` and `XXXXX` are the name and port number shown under [log destinations](https://papertrailapp.com/account/destinations).

# Alternatives: local log file or system syslog

If you do not want to have haproxy log directly to Papertrail, you can use one of these two methods instead:

* have haproxy log to a standalone text log file and transmit that to Papertrail. See [Configuring centralized logging from text log files in Unix with remote_syslog2](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix).
* have haproxy log to the system's syslog daemon and use that to transmit to Papertrail. See [Configuring remote syslog from Unix/Linux and BSD/macOS](/kb/configuration/configuring-remote-syslog-from-unixlinux-and-bsdos-x).
