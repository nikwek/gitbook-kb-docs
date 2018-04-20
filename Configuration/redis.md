---
layout: default
category: configuration
title: Redis
order: 157
---

# Redis

Aggregate logs from [Redis](https://redis.io), an open-source, in-memory data store. 

# Configuration

All logging settings are located in `/etc/redis/redis.conf`. Redis supports logging to Papertrail via two methods: syslog and local text log files. Redis logs have four levels of verbosity. To select a level, set `loglevel` to one of:

* `debug` (a lot of information, useful for development/testing)
* `verbose` (includes information not often needed, but logs less than debug)
* `notice` (moderately verbose, ideal for production environments)
* `warning` (only very important / critical messages are logged)

## Local Log File (using remote_syslog2)

By default, Redis logs to `/var/log/redis/redis-server.log`. Use a logging daemon like [remote_syslog2](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix/) to send these logs to Papertrail. 

## Syslog

To send Redis logs to Papertrail using the host's native syslog, [configure syslog](/kb/configuration/configuring-remote-syslog-from-unixlinux-and-bsdos-x/), then enable it within Redis by uncommenting the `syslog-enabled` line and setting it to `yes`.


The program name can be overridden from the default of `redis` by uncommenting and updating `syslog-ident`.
