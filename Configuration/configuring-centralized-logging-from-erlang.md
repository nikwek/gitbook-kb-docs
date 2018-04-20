---
layout: default
category: configuration
title: Configuring centralized logging from Erlang
sidebar_title: Erlang
order: 88
---

# Configuring centralized logging from Erlang apps

Papertrail can accept logs from any Erlang (BEAM) app, with or without the
`lager` logging library.

# Send log file with remote_syslog2

Configure your Erlang app to log to a file as usual, then 
[transmit the log file](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix) to Papertrail using [remote_syslog2](https://github.com/papertrail/remote_syslog2).
This does not require any modifications to the app.

# Send events from Erlang app

Alternatively, your app can send a log message directly to Papertrail using the
remote syslog protocol.

To do this, add [schlagert/syslog](https://github.com/schlagert/syslog) to
the app and point it at Papertrail. See
[Add Systems](https://papertrailapp.com/systems/setup) for `dest_host` and
`dest_port` log destination values. Use either protocol RFC; if in doubt,
choose `rfc5424`.

Since `schlagert/syslog` is emitting UDP, which is inherently connection-less,
the write calls are non-blocking.

# Send events with lager library and system syslog

Erlang applications that already use the `lager` logging library can
configure it to also log to syslog. See [lager_syslog](https://github.com/basho/lager_syslog), which will send logs
to a syslog daemon running on the local host (such as rsyslog or syslog-ng).
Follow [standard setup](https://papertrailapp.com/systems/setup) to
configure the syslog daemon to forward to Papertrail.
