---
layout: default
category: configuration
title: Configuring centralized logging from Node.js apps
sidebar_title: Node.js
order: 145
---

# Configuring centralized logging from Node.js apps

Papertrail can accept logs from any Node.js app using a regular text log file
or the libraries `winston` or `bunyan`. Here's how.

# Send log file with remote_syslog2

Have Node log to a file, then [transmit the log file](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix) to Papertrail using [remote_syslog2](https://github.com/papertrail/remote_syslog2).

To make Node log to a file, either redirect its output or use the [fs module](http://nodejs.org/docs/latest/api/fs.html#fs_fs_write_fd_buffer_offset_length_position_callback). For example, to redirect stdout to `myapp.log`, start the daemon with:

```shell
$ nohup node myapp.js > myapp.log &
```

# Send events from Node.js

Use the [winston Papertrail transport](https://github.com/kenperkins/winston-papertrail) to asynchronously transmit events from Node.js. See the [transport README](https://github.com/kenperkins/winston-papertrail).

To log unhandled exceptions, see the [winston documentation](https://github.com/flatiron/winston#handling-uncaught-exceptions-with-winston).

[node-bunyan](https://github.com/trentm/node-bunyan) users can
use the [node-bunyan-syslog](https://github.com/mcavage/node-bunyan-syslog#usage)
stream.

## Related

* winston [generic syslog](https://github.com/indexzero/winston-syslog) transport.
* [node-posix](https://github.com/melor/node-posix) library for sending to local system syslog daemon, which can then relay to Papertrail using standard [syslog instructions](/kb/configuration/configuring-remote-syslog-from-unixlinux-and-bsdos-x).
* [rconsole](https://github.com/tblobaum/rconsole) for logging Node console to local syslog.
