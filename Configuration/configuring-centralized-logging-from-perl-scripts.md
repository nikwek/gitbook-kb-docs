---
layout: default
category: configuration
title: Configuring centralized logging from Perl scripts
sidebar_title: Perl
order: 148
---

# Configuring centralized logging from Perl scripts

Papertrail can accept logs from any Perl script using either a local log file or the Sys::Syslog module.

# Send log file with remote_syslog2

Configure your script to log to a file as usual, then 
[transmit the log file](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix) to Papertrail using [remote_syslog2](https://github.com/papertrail/remote_syslog2).
This does not require any modifications to the app.

# Send events with Perl's Sys::Syslog module

Perl can also send log messages directly to Papertrail using >= 0.28 of the [Sys::Syslog](http://perldoc.perl.org/Sys/Syslog.html) module.

To determine which version of Sys:Syslog is installed:

```shell
$ perl -MSys::Syslog -e 'print "$Sys::Syslog::VERSION\n"'
```

To update it:

```shell
$ sudo perl -MCPAN -e shell
  # Accept all the defaults or modify as necessary
cpan> install Sys::Syslog
cpan> exit
```

## Example

The following sends a UDP syslog message:

```perl
use Sys::Syslog qw(:standard :macros setlogsock);  # standard functions & macros

$program = "PROGRAM";
$sender = "SENDER";
openlog("$program $sender", 'noeol,nonul');
setlogsock({ type => "udp", host => "logsN.papertrailapp.com", port => XXXXX });
syslog('info', 'something happened over here');
closelog();
```

Replace `logsN` and `XXXXX` with details from one of your
Papertrail [log destinations](https://papertrailapp.com/account/destinations), and `$program` and `$sender` as desired.
