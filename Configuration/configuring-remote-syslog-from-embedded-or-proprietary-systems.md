---
layout: default
category: configuration
title: Configuring remote syslog from embedded or proprietary systems
sidebar_title: Embedded devices
order: 48
---

# Configuring remote syslog from embedded or proprietary systems

# Introduction

To send from embedded devices, generate log messages in syslog format. Syslog is documented as [RFC 5424](http://tools.ietf.org/html/rfc5424) and [RFC 3164](http://www.ietf.org/rfc/rfc3164.txt).

Papertrail supports and automatically detects both formats. Papertrail also tries to extract as much meaning as possible from malformed messages.

Syslog is an extremely simple transport for logs of all kinds. While the RFCs provide a much more thorough explanation, at its most basic, each message is transmitted as a simple string. For example:

```
<22>Apr 25 23:45:56 sendername programname: the log message
```

At its most basic, this is even a valid message:

```
<22>sendername: the log message
```

# Format

## Choosing a format

Syslog has an older format (RFC 3164) and a newer format (RFC 5424). If you plan to send timestamps in [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) format, like `2014-06-18T09:56:21Z`, or are creating a new application or device, we recommend RFC 5424 format.

Papertrail also fully supports the older format, RFC 3164. If absolute minimalism is the goal, RFC 3164 is slightly simpler.

## RFC 5424 (newer)

```
<22>1 2014-06-18T09:56:21Z sendername programname - - - the log message
```

Replace the timestamp, sendername, programname, and of course the log message. The `<22>1 ` can be treated as a string literal and does not need to change. Note the space between the 1 and the start of the timestamp.

## RFC 3164 (older)

Generate a message like this:

```
<22>Apr 25 23:45:56 sendername programname: the log message
```

Replace the timestamp, sendername, programname, and of course the log message.

Consistent with the RFC, the timestamp and program/component name (syslog "tag" field) are optional fields, as is the PID (not shown). We recommend including the timestamp and program/component name, but omitting the PID.

## What is the `<22>`?

As the RFCs explain, the messages should include a body and a valid numeric facility/severity (syslog "priority" field), which is "22" in the examples above. While you can choose to generate other values, for most integrations, using only `<22>` works fine.

# Log viewer

In Papertrail's [Event viewer](https://papertrailapp.com/events), the sender name and program/component name become clickable orange and blue links to see [surrounding context](/kb/how-it-works/event-viewer#context). Used well, this is a powerful way to see similar logs.

For example, the message:

```
<22>1 2014-06-18T09:56:21Z sendername programname - - - the log message
```

Is displayed like this in Papertrail's log viewer:

![embedded-example.png](/assets/images/embedded-example_normal.png)

# Network protocol

Transmit messages via either:

* UDP
* TCP with TLS
* Plaintext TCP

See [Self-service protocol options](http://blog.papertrailapp.com/self-service-protocol-options/) for more on selecting a protocol.

Send to the hostname & destination port provided by Papertrail.

A quick-and-dirty way to do this is to use netcat. The exact syntax varies depending on the flavour, but when using the OpenBSD variant, the following will submit a test message to Papertrail over UDP:

```shell
$ echo test | nc -w0 -u logsN.papertrailapp.com XXXXX
```

where `logsN` and `XXXXX` are the log host and port number shown under [log destinations](https://papertrailapp.com/account/destinations).

Your embedded software vendor or OEM may have additional documentation on its syslog support.  We welcome inquiries about other devices or new logging implementations.
