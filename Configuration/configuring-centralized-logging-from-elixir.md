---
layout: default
category: configuration
title: Configuring centralized logging from Elixir
sidebar_title: Elixir
order: 85
---

# Configuring centralized logging from Elixir apps

Papertrail can accept logs from any Elixir app, with or without the
`logger_papertrail_backend` logging library.

# Send log file with remote_syslog2

Configure your Elixir app to log to a file as usual, then 
[transmit the log file](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix) to Papertrail using [remote_syslog2](https://github.com/papertrail/remote_syslog2).
This does not require any modifications to the app.

# Send events from Elixir app

Alternatively, the app can send a log message directly to Papertrail using the
syslog protocol.

To do this, add [logger_papertrail_backend](https://github.com/larskrantz/logger_papertrail_backend) to
the app and point it at Papertrail. See
[Add Systems](https://papertrailapp.com/systems/setup) for the host and `<port>` log destination values.

Since `logger_papertrail_backend` is emitting UDP, which is inherently connection-less,
the write calls are non-blocking.
