---
layout: default
category: configuration
title: Configuring remote syslog from Unix/Linux and BSD/macOS
sidebar_title: Unix & BSD system logs
order: 0
---

# Configuring remote syslog from Unix/Linux and BSD/macOS

To log from a Unix system, edit the system's syslog daemon config file. These instructions are a reference. Papertrail will provide more specific instructions (including a log destination) when you [add a system](https://papertrailapp.com/systems/setup).

These instructions will typically pick up operating system logs. Some apps write logs directly to text files, bypassing the syslog daemon. To collect logs from these apps, use [remote_syslog2](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix/) instead.

# Determine system logger

See which logger your system uses. Run:

```
ls -d /etc/*syslog*
```

Which filename is listed? <a href="#rsyslog">rsyslog.conf</a>, <a href="#syslog-ng">syslog-ng.conf</a>, <a href="#syslog">syslog.conf</a>, or <a href="#none">none</a>.

# rsyslog.conf

rsyslog is often seen on: Debian; Fedora; SuSE; Ubuntu; most other Linux distributions.

## Configure rsyslog

As root, edit /etc/rsyslog.conf or /etc/syslog.conf with a text editor (like pico or vi). Paste a line like this at the end of the file:

```
*.*                       @logsN.papertrailapp.com:XXXXX
```

Replace `logsN` and `XXXXX` with the host and port from
Papertrail's Web interface. (seen on [Add Systems](https://papertrailapp.com/systems/setup) or [Log Destinations](https://papertrailapp.com/account/destinations)).

## Activate change

Tell rsyslog to activate the change (on most OSes):

```shell
$ sudo /etc/init.d/rsyslog restart
```

On Ubuntu:

```shell
$ sudo service rsyslog restart
```

Log messages should begin appearing in Papertrail. Optionally, configure [encrypted logging with TLS](/kb/configuration/encrypting-remote-syslog-with-tls-ssl#rsyslog).

By default, rsyslog sends messages from the system's hostname (such as `www42`). To change this behavior and choose your own hostname or use the FQDN, see [How can I override the hostname?](/kb/configuration/advanced-unix-logging-tips#set_hostname).

<a name="syslog-ng"></a>

# syslog-ng.conf

syslog-ng is often seen on: Gentoo 2005.0+; SuSE 9.3+.

## Configure syslog-ng

As root, edit /etc/syslog-ng.conf with a text editor. Find a line starting with source. For example: `source s_sys {..}`.

At the end of the file, paste this configuration. Replace `s_sys` with the source name above, typically `s_sys`, `src`, `s_all`, or `s_local`:

```
destination d_papertrail {
    udp("logsN.papertrailapp.com" port(XXXXX));
};

# replace "s_sys" with the name you found:
log { source(s_sys); destination(d_papertrail); };
```

Replace `logsN` and `XXXXX` with the host and port from Papertrail's Web interface (seen on [Add Systems](https://papertrailapp.com/systems/setup) or [Log Destinations](https://papertrailapp.com/account/destinations)).

## Activate change

Tell syslog-ng to activate the change:

```shell
$ sudo killall -HUP syslog-ng
```

Log messages should begin appearing in Papertrail. Optionally, configure [encrypted logging with TLS](/kb/configuration/encrypting-remote-syslog-with-tls-ssl#syslog-ng).

<a name="syslog"></a>

# syslog.conf

syslogd and sysklogd are often seen on: BSDs; CentOS; Gentoo 2004.3 and older; Mac macOS; RHEL; Slackware; Solaris; most other Unices.

[remote_syslog2](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix) can be used in lieu of syslogd.

Some versions of syslog do not support custom ports and must use the default port 514, but modern BSD versions (including macOS) support custom ports.

## Default port (514)

{% include "../\_includes/add_systems_port_514_navigation_steps.md" %}

Then, follow the additional instructions to configure the daemon.

## Custom port

### Configure syslogd

As root, edit /etc/syslog.conf with a text editor (like pico or vi). Paste this line at the end of the file:

```
*.*                                @logsN.papertrailapp.com:XXXXX
```

replacing `logsN` and `XXXXX` with the values from the [log destination](https://papertrailapp.com/account/destinations).

### Activate change

Tell syslog to activate the change (on most OS's):

```shell
$ sudo killall -HUP syslog syslogd
```

Log messages should begin appearing in Papertrail.

# Test (optional)

To confirm messages are being sent and received, you can generate a test message by running:

```shell
$ logger "Testing Papertrail message delivery"
```

The test message should appear nn the [event viewer](https://papertrailapp.com/events) almost immediately. If it doesn't arrive, try sending a [standalone test message](/kb/configuration/troubleshooting-remote-syslog-reachability#test-system-logging-config).

# No syslog configuration found

If `ls -d /etc/*syslog*` did not find any matching files, try these:

* On Fedora Linux 20 and later, install the rsyslog package ([why?](https://fedoraproject.org/wiki/Changes/NoDefaultSyslog)). Run: `sudo yum install rsyslog`
* On other Linux distributions and Unix variants other than Linux, try looking for files containing `syslog` outside of `/etc/`. Run: `sudo find / -name "*syslog*" -print`
* Ask us. We've probably seen it.

# Logs not appearing?

The most common cause is a local or external firewall blocking outbound UDP traffic.
Solve this by adding an allow rule based on the port number shown on 
[Log Destinations](https://papertrailapp.com/account/destinations).

For more generic troubleshooting information, see
[Troubleshooting remote syslog reachability](/kb/configuration/troubleshooting-remote-syslog-reachability/).
