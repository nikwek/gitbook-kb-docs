---
layout: default
category: configuration
title: Configuring centralized logging from text log files in Unix (remote_syslog2)
sidebar_title: Unix & BSD text log files (remote_syslog2)
order: 10
---

# Configuring centralized logging from text log files in Unix with [remote_syslog2](https://github.com/papertrail/remote_syslog2)

Use `remote_syslog2` to aggregate logs from any text file, like app log files, in Unix and BSD.

# Why remote_syslog2

To send log files to Papertrail from applications and daemons that don't support syslog, run Papertrail's tiny standalone [remote_syslog2](http://github.com/papertrail/remote_syslog2) daemon. It tracks one or more log files and sends new entries to Papertrail in realtime.

`remote_syslog2` works for any text log files, has no impact to or reliance on the system syslog daemon or its logging configuration, is easy to set up, and forwards logs directly to Papertrail. No adjustments to `syslog.conf`, `rsyslog.conf`, or `syslog-ng.conf` are required.

Some applications that typically log to text files, such as [MySQL](/kb/configuration/configuring-centralized-logging-from-mysql-query-logs), [Apache](http://www.oreillynet.com/pub/a/sysadmin/2006/10/12/httpd-syslog.html), and Tomcat ([log4j](http://www.mail-archive.com/log4j-user@logging.apache.org/msg06907.html)), have internal support for logging directly to syslog. For apps like these, either their internal syslog or `remote_syslog2` can be used to send logs to Papertrail.

# Setup

## Automated (Configuration Management)

`remote_syslog2` can be deployed and configured with configuration management tools. Papertrail provides official support for Chef, Puppet, and Salt:

* [chef-papertrail](https://github.com/papertrail/chef-papertrail)
* [puppet-papertrail](https://github.com/papertrail/puppet-papertrail)
* [salt-papertrail](https://github.com/papertrail/salt-papertrail)

These configuration management modules only configure and deploy `remote_syslog2`. To configure and manage the system's syslog daemon configuration, see [Configuring remote syslog from Unix/Linux and BSD/macOS](/kb/configuration/configuring-remote-syslog-from-unixlinux-and-bsdos-x/).

## Manual

REMOVED INCLUDE

<a name="problem"></a>

# Troubleshooting

<div class="alert alert-info" role="alert">
  <div class="fa fa-info-circle alert-icon"></div>
  <div class="alert-message"><code>remote_syslog2</code> is daemonized as <code>remote_syslog</code>. When you're looking for the process, look for <code>remote_syslog</code> (not <code>remote_syslog2</code>).</div>
</div>

## The system rebooted and `remote_syslog` didn't start

Install an [init file](https://github.com/papertrail/remote_syslog2#auto-starting-at-boot).

## Logs not appearing?

Please feel free to drop a mail into <support@papertrailapp.com>, either instead of following these instructions or while trying them. We enjoy helping, and these steps are only here if they let you save time by troubleshooting independently.

There are a few reasons `remote_syslog` might not be sending logs.

### Is `remote_syslog` running? 

Check with `ps auxww | grep [r]emote_syslog`. Exactly 1 process should be shown, like this:

```
root     24501    0.0 0.4    13952 8864 ? S   Mar01 2:45 remote_syslog
```

### Does it have the correct access permissions?

In the example above, the process is running as user `root`. If a user other than `root` is shown, does that user running `remote_syslog` have permission to write to `/var/run/`? `/var/run/` is the default location for PID files ([background](http://stackoverflow.com/questions/688343/reference-for-proper-handling-of-pid-file-on-unix)).

If `remote_syslog` isn't already running as root, try running it as root to test (such as with `sudo`), or specify an alternate location for the PID file (such as with `remote_syslog --pid-file=/tmp/some.pid`).

### Did it stop running?

If you can start `remote_syslog` but it subsequently stops running, try leaving it attached to the terminal (rather than daemonizing) using `remote_syslog -D`.

### Is it running, but suddenly stopped sending data?

...perhaps across many systems at the same time? This could be a firewall policy change or [log rotation](https://github.com/papertrail/remote_syslog2#log-rotation). However, `remote_syslog` plays well with all common log rotation systems (with no changes to the configuration of either), so we'd like to hear about this problem.

### Is `remote_syslog` monitoring log files that are stored on an NFS share? 

If so, enable polling via the `--poll` switch.

## Detailed troubleshooting

The next step is to see what `remote_syslog` is actually doing. Typically that's with [strace](http://en.wikipedia.org/wiki/Strace), such as:

```shell
$ strace -tt -s 500 -fp 12345
```

...where `12345` is the process ID of `remote_syslog`, obtained from the second column of `ps auxww`. This will output every call that it makes. We suggest sending the strace output to a file:

```shell
$ strace -tt -s 500 -fp 12345 -o strace.log
```

Feel free to send the `strace.log` file to us via <support@papertrailapp.com>. We'll look for a few things. Did the OS notify `remote_syslog` of writes to the log files? Did `remote_syslog` call `sendto()` (UDP) or `connect()` and `send()` (TCP) to try sending the message? What happened?

# Extra

See other options with:

```shell
$ remote_syslog --help
```

See also [Troubleshooting reachability](/kb/configuration/troubleshooting-remote-syslog-reachability/).

# Question?

Can we help? [Just ask](/).
