---
layout: default
category: configuration
title: Configuring centralized logging from MySQL query logs
sidebar_title: MySQL
order: 135
---

# Configuring centralized logging from MySQL query logs

Aggregate logs from MySQL, including slow queries and error messages.



# Introduction {#intro}

We recommend Papertrail's tiny standalone `remote_syslog2` daemon to read the `mysql-slow.log` (and any other log files) in realtime ([remote_syslog2](#remote_syslog2)). It requires no changes to MySQL, is easier to setup, and works for non-MySQL log files as well.

MySQL 5.1.20 and later can also use its own native syslog support plus the system-wide syslog daemon ([Native](#native)). To see which version of MySQL is installed, run: `mysql -V`.

The native method is not available in MySQL 5.1.19 and earlier.

# Send log file with remote_syslog2 {#remote_syslog2}"

REMOVED INCLUDE

# Send events from MySQL native syslog {#native}

MySQL versions 5.1.20 and later support logging to syslog natively.  Two arguments to mysqld_safe control logging: `--syslog` and `--log-error="/path/to/file"`

Both, either, or neither argument can be used.  mysqld_safe is generally started from `/etc/init.d/mysqld`.  Edit that file (or its equivalent for your distribution) and confirm that `--syslog` is included on the command line, or add it.

Here is a sample that sends MySQL logs to syslog and directly to a file:

```shell
$ /usr/bin/mysqld_safe --datadir="$datadir" --socket="$socketfile" \
    --syslog --log-error="/var/log/mysql.err" --pid-file="$mypidfile" \
    >/dev/null 2>&1 &
```

# More

Official documentation: [MySQL 5.1: The Error Log](http://dev.mysql.com/doc/refman/5.1/en/error-log.html), [MySQL 5.0: The Error Log](http://dev.mysql.com/doc/refman/5.0/en/error-log.html)
