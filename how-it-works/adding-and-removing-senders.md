---
layout: default
category: how-it-works
title: Adding and removing senders
sidebar_title: Managing senders
order: 27
---

# Adding and removing senders

# Adding a sender

By default, Papertrail log destinations accept logs from new senders and create the new sender name automatically (see [How are senders named?](#how-are-senders-named)). Adding a sender is as simple as <a data-trigger="configuration" href="#configuration">configuring logging</a> for a new machine or app.

In some cases, this relatively open default policy might not be the best fit, so Papertrail also provides more controlled options. Choose the balance between security and flexibility that best fits your environment.

## Require registration of new senders

For each [log destination](https://papertrailapp.com/account/destinations), disable auto-detection by unchecking **Yes, recognize logs from new systems** to make Papertrail silently drop messages sent from system names that don't already exist.

To register new systems after auto-recognition is disabled, either enable auto-recognition long enough to send a single message, then disable auto-recognition again, or use [papertrail-add-system](https://github.com/papertrail/papertrail-cli#addremove-systems-create-group-join-group) (or the [corresponding HTTP API call](/kb/how-it-works/settings-api#systems)) to register the system(s).

## Configure a random sender identifier

For environments where integrity is critical or where hostnames are publicly known, Papertrail can match messages against a value other than the sender's hostname, such as an assigned random string.

For example, here's how to tell Papertrail that the sender named `www42` will send with this random string as the syslog hostname:

```shell
$ papertrail-add-system --hostname C9M-0t3NxZ2XlpBS-y8upepeS1zNurT -s www42
```

Papertrail will show the system's hostname, `www42`, but its messages must contain the `C9M...` string as the hostname. This string can be used with `remote_syslog2` ([example](https://github.com/papertrail/remote_syslog2#override-hostname)), `rsyslog` ([example](/kb/configuration/advanced-unix-logging-tips/#set_hostname)), and most other senders.

Typically, the combination of the system hostname and the account-specific log destination is unique enough that using a separate random string as an identifier isn't required.

## Use source IPs

Alternatively, on [Add Systems](https://papertrailapp.com/systems/new), select _My syslogd only uses the default port_ and then provide the IP of each sender. 

## Disable UDP 

Optionally, you may also wish to ensure that every sender has gone through the TCP three-way handshake. To do so, disable UDP logging on the [log destination](https://papertrailapp.com/account/destinations).

# How are senders named?

Log senders like [rsyslog](/kb/configuration/configuring-remote-syslog-from-unixlinux-and-bsdos-x/) and [remote_syslog2](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix/) typically set a sender
identifier field in each syslog packet to the [system hostname](http://tools.ietf.org/html/rfc3164#section-4.1.2), though
it can be set to other values (see [Override the hostname sent by a logger](/kb/configuration/advanced-unix-logging-tips/#set_hostname)).

Because Papertrail accepts inbound links that use the sender name, such
as `https://papertrailapp.com/systems/www42`, the sender name must be
unique. When Papertrail receives a log message from a new sender and:

* has been configured to accept messages from unrecognized senders (see [Destinations](https://papertrailapp.com/account/destinations)), and
* the sender name in the log message is already in use by an existing sender

Papertrail will append a hyphen and sequence number (`-1`) to the default
sender name shown in Papertrail. For example: `www42-1`

This display name in Papertrail can still be edited, but it ensures that
administrators do not confuse the new sender with an existing sender.

# Removing senders

## Automatically removing senders

If **Automatically remove idle senders?** is checked for a [log destination](https://papertrailapp.com/account/destinations), idle senders will be removed two days after their most recent log message is no longer searchable, or one week after they’ve stopped sending, whichever is longer.

## Manually removing a sender

If **Automatically remove idle senders?** (in [Log Destination](https://papertrailapp.com/account/destinations) settings) is **not** checked for a given destination, you'll need to manually remove any sender that's no longer needed.

To remove a sender:

1. Navigate to your [Dashboard](https://papertrailapp.com/dashboard).
2. Choose the **All Systems** group link.
3. Find the system in the group list, and click the **Settings** button to the right.

![Sender Settings Button](/assets/images/list_of_senders_with_arrow_to_settings.png){: width="877"}

* At the bottom right of the settings page, click **Delete this system**.

![Delete This System](/assets/images/delete_this_system.png){: width="252"}
