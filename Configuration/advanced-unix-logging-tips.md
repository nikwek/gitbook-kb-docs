---
layout: default
category: configuration
title: Advanced Unix logging tips
order: 43
---

# Advanced Unix logging tips

Occasionally Asked Questions.

If you're just getting started and not already familiar with logging, start on [Setup](https://papertrailapp.com/systems/setup), not this document.

# Contents

**Common tasks**

* [Override the hostname sent by a logger](#set_hostname)
* [Import an existing log file to Papertrail](#import)
* [Aggregate output from a command or cron job](#pipe)
* [Set up a loghost that relays to Papertrail](#loghost)

**rsyslog**

* [Tweak queue options for connection failure](#rsyslog_queue)
* [Aggregate local log files with rsyslog](#rsyslog_aggregate_log_files)
* [Some messages from rsyslog are attributed to an IP address](#rsyslog_last_message_repeated)
* [A Xen/EC2 hostname is `domU-..` when a new system boots (then changes to the hostname I've configured)](#rsyslog_hostname_restart)
* [Tell rsyslog to drop messages from certain syslog facilities](#rsyslog_drop_messages)
* [rsyslog not transmitting anything or logging to disk](#rsyslog_not_transmitting)
* [ANSI color escape sequences appear as strings, not colors](#ansi)

**FreeBSD**

* [`syslogd` is not transmitting anything](#syslogd_silent)

# Common tasks

<a name="set_hostname"></a>

## Override the hostname sent by a logger

Papertrail honors the hostname sent by your log sender, such as `rsyslog` or `remote_syslog2`. By default, this is the system hostname. You can always rename a sender within Papertrail's Web interface by visiting the [Dashboard](http://papertrailapp.com/dashboard), clicking **All Systems**, then clicking **Edit Settings** for the sender. Changing the display name does not affect event attribution.

However, some systems may have hostnames like `ip-23-56-78-12` or `domU-49835` that aren't very meaningful. You can explicitly configure the log sender to use a hostname of your choosing, and its messages will appear in Papertrail under that system name. Here's how.

### rsyslog

To change the hostname rsyslog sends, add the following directive as the very first line in `/etc/rsyslog.conf` before any modules are loaded:

```
$LocalHostName yourhostname
```

Alternatively, to have rsyslog send with the fully-qualified domain name (FQDN, such as `system1.example.com`) instead of simply the hostname (`system1`), use the directive:

```
$PreserveFQDN on
```

This is rarely needed. We recommend using the hostname (without the domain name) unless you have identically-named systems in different domains.

An alternative that allows sending different logs as different hostnames is to set a custom template:

```
$template MyTemplate, "<%pri%> %timestamp% MySpoofedHostName %syslogtag% %msg%\n"
$ActionForwardDefaultTemplate MyTemplate
```

### syslog-ng

Add the following above the `destination` stanza that sends to Papertrail:

```
template t_papertrail {
  template("<$PRI>$DATE yourhostname $PROGRAM: $MSG\n");
};
```

... where `yourhostname` is the hostname or IP that you want Papertrail to recognize this system as.

In the default template format, syslog-ng uses `$HOST` (the system hostname). This is effectively replacing a passthrough option with a static string of your choosing.

Apply it by adding the template directive to the `tcp` or `udp` options inside the `destination` stanza:

```
destination d_papertrail {
  udp("logsN.papertrailapp.com" port(XXXXX) template(t_papertrail) );
};
```

### remote_syslog2

To change the hostname remote_syslog2 sends from, pass the `--hostname` option on the command line:

```shell
$ remote_syslog --hostname=yourhostname
```

or use the configuration directive in the log_files.yml:

```yaml
hostname: yourhostname
```

## Aggregate output from a command or cron job

To capture output from a Unix command or a cron job, pipe it to the standalone Unix program `logger`. Anything piped to `logger` goes to the system syslog daemon and thus on to Papertrail (as long as syslog is sent to Papertrail).

Example:

```shell
$ echo "hi" | logger
```

You can also choose a tag for the logs, like a program name. For example:

```shell
$ uptime | logger -t uptime
$ uptime | logger -t bob-monitoring
```

Any Unix command can work this way, including cronjobs. Most cronjobs redirect standard error (file descriptor 2) to standard out (file descriptor 1). By default, `cron` emails it to you, but it can also be sent to `logger`:

```shell
$ some-command 2>&1 | logger
```

or tagged as `cron`:

```shell
$ some-command 2>&1 | logger -t cron
```

## Set up or use a loghost that relays to Papertrail

Papertrail can receive logs directly from the originating systems and apps, or
relayed through a local log host. Many syslog daemons can relay messages
without modifying the syslog header. Because the relay does not modify the
header, Papertrail will automatically attribute the message to the original
sender, not the relay.

Here's how to use `rsyslog` as a local aggregator that relays to Papertrail.

### Installation

1. Install a Linux distribution such as [CentOS](http://centos.org/modules/tinycontent/index.php?id=15). This can be a tiny (256 MB) virtual machine on minimal hardware. See [CentOS downloads](http://wiki.centos.org/Download). If prompted, choose to install without graphic (X Window System) support.
2. Once booted, login as root and confirm that `/etc/rsyslog.conf` exists by running: `ls -la /etc/rsyslog.conf`
3. Edit `/etc/rsyslog.conf` and uncomment these 4 lines:

```
$ModLoad imudp.so
$UDPServerRun 514
$ModLoad imtcp.so
$InputTCPServerRun 514
```

4. Allow systems to log to your system. You may want to restrict this to only expected source IPs; adjust according to your network topology.

```shell
$ sudo iptables -A INPUT -m state –state NEW -m udp -p udp –dport 514 -j ACCEPT
$ sudo iptables -A INPUT -m state –state NEW -m tcp -p tcp –dport 514 -j ACCEPT
$ sudo service iptables save
```

5. Configure other systems and apps to log to this new system. Feel free to use Papertrail's instructions, but instead of configuring them to log directly to Papertrail (such as `logsN.papertrailapp.com:XXXXX`), point them at this system (such as `myloghost.example.com:514` or `192.168.1.42:514`).

### Configuration

1. Configure rsyslog according to Papertrail's standard instructions on [Add Systems](https://papertrailapp.com/systems/setup). The same configuration will relay not only logs from this system, but also logs received from other systems.
2. Restart rsyslog: `$ sudo service rsyslogd restart`.

As soon as a system generates a new message, it will be visible in Papertrail with the original system's name.

<a name="import"></a>

## Import an existing log file to Papertrail

This is almost never needed and very rarely leads to a great result. Two cases where it may help: 

* You're brand new to Papertrail and absolutely need existing logs.
* You discover a file that wasn't being aggregated and is absolutely required.

There are two relatively easy ways to import existing log messages to Papertrail. Timestamps in the log file are not honored but the message contents will be searchable.

### remote_syslog2

Run [remote_syslog2](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix) and point it at a temporary file, then use `cat` to append the existing log file contents to that temp file. Here's an example where `/tmp/placeholder` is the temp file:

```shell
$ touch /tmp/placeholder
$ remote_syslog -D -d logsN.papertrailapp.com -p XXXXX /tmp/placeholder
$ cat /home/httpd/logs/missing-file.log >> /tmp/placeholder
```

Then wait a minute or two so `remote_syslog2` can send the newly-appended logs (depending on how large the file is). Once you see them in Papertrail, kill `remote_syslog2`.

### syslog daemon

If your system is transmitting syslog (like from `rsyslog` or `syslog-ng`), you can "piggyback" on that and feed a text file into syslog. The Unix `logger` command accepts a filename and sends each line to the system syslog daemon. Here's an example:

```shell
$ logger -f the-missing-file.log
```

Again, this is very rarely needed.

# rsyslog

<a name="rsyslog_queue"></a>

## Tweak queue options for connection failure

This section is irrelevant for those using the default settings, which log via UDP. There is no concept of receipt verification or connections.

If you have set up [TCP with TLS](/kb/configuration/encrypting-remote-syslog-with-tls-ssl), rsyslog is aware of whether it can connect to and transmit to Papertrail. If it can't, it needs to know what to do with incoming syslog messages. This can be to drop them or to queue them in memory or in memory and on disk.

While a full discussion of `rsyslog` [queues](http://www.rsyslog.com/doc/queues.html) is outside the scope of this document, here's a sample configuration that queues up to 100,000 lines in memory and on disk, tries to reconnect to Papertrail in perpetuity, and times out new `syslog()` calls in 2ms (so that your apps do not block if the queue fills). This would go immediately before the `*.* @@logsN.papertrailapp.com` line in your rsyslog config:

```
$ActionResumeInterval 10
$ActionQueueSize 100000
$ActionQueueDiscardMark 97500
$ActionQueueHighWaterMark 80000
$ActionQueueType LinkedList
$ActionQueueFileName papertrailqueue
$ActionQueueCheckpointInterval 100
$ActionQueueMaxDiskSpace 2g
$ActionResumeRetryCount -1
$ActionQueueSaveOnShutdown on
$ActionQueueTimeoutEnqueue 2
$ActionQueueDiscardSeverity 0
```

<a name="rsyslog_aggregate_log_files"></a>

## Aggregate local log files with rsyslog

The easiest way to aggregate local log files is with Papertrail's tiny standalone [remote_syslog2](https://github.com/papertrail/remote_syslog2) daemon, which is documented [here](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix).

Papertrail also works great with `rsyslog` or `syslog-ng` collecting local log files. Most rsyslog configs use `/etc/rsyslog.conf` to load additional files from `/etc/rsyslog.d/`. If you have this directory (see `ls /etc/rsyslog.d/`), create `/etc/rsyslog.d/95-papertrail.conf` with the Papertrail config. The `95` means that the config will be loaded near the end. If the `/etc/rsyslogd.d/` directory does not exist, append the config to `/etc/rsyslog.conf` instead.

Here's an example annotated `rsyslog.conf` snippet which will transmit one or more log files in realtime:

```
# load module to read from local files
$ModLoad imfile

# use a non-default ruleset (keeps logs out of /var/log/)
$RuleSet papertrail
$InputFileBindRuleset papertrail

# for each local log file path, duplicate the 5 lines below and edit lines 1-3
$InputFileName /path/to/some.log
$InputFileTag somelog:
$InputFileStateFile papertrail-somelog
$InputFilePersistStateInterval 100 # update state file every 100 lines
$InputRunFileMonitor

# destination (see https://papertrailapp.com/systems/setup)
*.* @logsN.papertrailapp.com:XXXXX
# for clarity, explicitly discard everything (this is typically not necessary)
*.* ~

# all done. change to default ruleset (RSYSLOG_DefaultRuleset) for any following config
$InputFileBindRuleset RSYSLOG_DefaultRuleset
$RuleSet RSYSLOG_DefaultRuleset
```

Finally, inform rsyslog of the change with `sudo /etc/init.d/rsyslog restart`. We also recommend explicitly defining a `WorkDirectory` directive; see below.

### State files (WorkDirectory)

`rsyslog` uses state files to track which parts of a log file have already been transmitted. By default, these state files are written in a directory chosen by the `WorkDirectory` option. `rsyslog` needs permission to write to that directory. To see the directory, run:

```shell
$ grep -r WorkDirectory /etc/*rsyslog*
```

Many distributions have `rsyslog` drop to a non-root user. For example, Ubuntu uses the `syslog` user. See which user `rsyslog` runs as with `ps auxw | grep rsyslog | grep -v grep`. Confirm that the WorkDirectory is writable by that user.

If you don't find any explicit configuration lines, `rsyslog` may attempt to use `/rsyslog/work/` or another path chosen by the rsyslog package maintainer. This directory probably doesn't exist and wouldn't be the best place for spool files if it did. To change to another directory, add this configuration option:

```
$WorkDirectory /var/run
```

### RuleSets

The configuration above uses RuleSets to explicitly keep messages from local files out of the OS syslog. If your `rsyslog` version doesn't support the `RuleSet` command, remove both `RuleSet` commands from the config stanza above and ensure that the Papertrail config stanza is last in the config file.

rsyslog configs are processed sequentially, so OS syslog messages will have already been written by the time that these files are read in.

<a name="rsyslog_last_message_repeated"></a>

## Some messages from rsyslog are attributed to an IP address 

This most often occurs with `Last message repeated...` messages.

This is a bug in rsyslog which was [fixed](http://lists.adiscon.net/pipermail/rsyslog/2012-June/030162.html) on June 11, 2012. Either upgrade rsyslog, ignore the relatively few `Last message repeated n times` messages which are attributed to IPs, or set `$RepeatedMsgReduction off` to disable these messages entirely (at the expense of potentially higher output).

<a name="rsyslog_hostname_restart"></a>

## A Xen/EC2 hostname is `domU-...` when a new system boots (then changes to the configured hostname)

The `rsyslogd` daemon obtains the system hostname at the time it starts, and if a different hostname is configured, it's added later. This can lead to initial boot log messages appearing under the machine-chosen hostname like `domU-11-22-33-44-55-66`.

A few minutes or a few hours later, the log hostname changes to the one you configured. The change often happens the following evening when `rsyslogd` is notified of a log rotation and detects the new hostname.

There are a few simple ways to handle this:

* Ignore it.
* Tell `rsyslogd` about the hostname change by restarting it after the hostname is changed (typically during the boot process, EC2 bootstrap process, or initial configuration management invocation).
* As a last resort, use `rsyslog`'s `LocalHostname` option to explicitly define the hostname. [Here's how](#set_hostname). This works best in environments where the rsyslog config file is already generated by a configuration management system like puppet or chef (which is aware of the hostname).

<a name="rsyslog_drop_messages"></a>

## Tell rsyslog to drop messages from certain syslog facilities

To exclude all messages from a specific facility, change `*.*` to include a statement that drops messages in that facility. For example, to drop messages in the facility called `cron`, use `cron.none`. This will tell `rsyslog` to send all messages except `cron`:

```
*.*;cron.none             @logsN.papertrailapp.com:XXXXX
```

More: [facilities](http://en.wikipedia.org/wiki/Syslog); [Google query](http://www.google.com/search?q=cron.none)

<a name="rsyslog_not_transmitting"></a>

## `rsyslog` not transmitting anything or logging to disk

If new messages are not appearing in the local log files, and no messages are being transmitted to Papertrail, edit `/etc/logrotate.d/rsyslog` and change:

```
postrotate
  reload rsyslog >/dev/null 2>&1 || true
endscript
```

To:

```
create 640 syslog adm
postrotate
  /sbin/restart rsyslog >/dev/null 2>&1 || true
endscript
```

This will restart the `rsyslog` daemon rather than reloading it, and ensure that new log files are created with the correct permissions and ownership.

Solution originally posted on [Launchpad](https://bugs.launchpad.net/ubuntu/+source/rsyslog/+bug/940030)

<a name="ansi"></a>

## ANSI color escape sequences appear as strings, not colors

By default, `rsyslog` escapes ANSI color codes prior to sending them to
Papertrail. If your logs include ANSI color and traverse `rsyslog`, add this
configuration directive near the top of `rsyslog.conf`:

```
$EscapeControlCharactersOnReceive off
```

Restart `rsyslog` with `sudo service rsyslog restart`. Future log messages should
be transmitted with actual escape sequences and thus be
[colorized](/kb/how-it-works/log-colorization/) by Papertrail.

# FreeBSD

<a name="syslogd_silent"></a>

## `syslogd` is not transmitting anything

FreeBSD often is configured with `syslogd` set to both ignore incoming syslog messages and silently refuse to send outgoing ones. To see whether FreeBSD is configured this way, run:

```shell
$ ps auxw | grep syslog | grep -v grep
```

If you see a line with the argument `-ss` (two `s` letters), then `syslogd` will neither accept nor transmit log messages. For example:

```
root  50908   0.0  0.0  12184   1400  ??  Ss   12:48PM     0:00.01 /usr/sbin/syslogd -ss
```

To change this, edit `/etc/rc.conf` and remove one of the `s` letters so that it is started with `syslogd -s` (one `s`), then restart `syslogd`.
