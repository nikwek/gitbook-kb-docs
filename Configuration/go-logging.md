---
layout: default
category: configuration
title: Go logging
sidebar_title: Go
order: 88
---

# Go logging

# Introduction

The [Go Programming Language](https://golang.org/) or `golang` can log to Papertrail via three methods.

# Send log file with remote_syslog2

Log to a text log file(s), then 
[transmit the log file](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix) to Papertrail using [remote_syslog2](https://github.com/papertrail/remote_syslog2).

# Send events from the Go app

Have the Go app transmit logs to Papertrail with the standard 
[syslog package](https://golang.org/pkg/log/syslog/) or a fork like 
[srslog](https://medium.com/@sirsean/srslog-sending-syslog-messages-from-go-a270d9c74ecd). 

Call [syslog.Dial](https://golang.org/pkg/log/syslog/#Dial) with the
hostname and port [provided](https://papertrailapp.com/account/destinations) by Papertrail. 
For example:

```go
import "log/syslog"

w, err := syslog.Dial("udp", "logsN.papertrailapp.com:XXXXX", LOG_EMERG | LOG_KERN, "myapp")
if err != nil {
    log.Fatal("failed to dial syslog")
}
w.Info("Any log message")
w.Err("Another log message")
```

# Send events with the system syslog

Use the Go [syslog package](https://golang.org/pkg/log/syslog/) to send logs to a local 
syslog daemon ([example](http://technosophos.com/2013/09/14/using-gos-built-logger-log-syslog.html)).
Have the local syslog daemon transmit the logs to Papertrail
using Papertrail's [Unix/Linux syslog instructions](/kb/configuration/configuring-remote-syslog-from-unixlinux-and-bsdos-x/).
