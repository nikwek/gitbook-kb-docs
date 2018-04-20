---
layout: default
category: configuration
title: log4net logging
sidebar_title: C# & .NET (log4net)
order: 70
---

# log4net logging

# Introduction

[log4net](http://logging.apache.org/log4net/) is "a tool to help the programmer output log statements to a variety of output targets. log4net is a port of the excellent Apache log4j™ framework to the Microsoft® .NET runtime."

Papertrail supports aggregating messages from a native log4net appender, providing a live searchable log console for your .NET app.

{% include "../\_includes/simplesyslog.md" %}

# Installation

Install [log4net](http://logging.apache.org/log4net/release/manual/configuration.html) in your app. At its core, this is importing the classes:

```csharp
using log4net;
using log4net.Config;
```

and then initializing log4net by pointing it to a config file path (which we'll create in the next step):

```csharp
XmlConfigurator.Configure(new System.IO.FileInfo(args[0]));
```

In the example above, the path to the config file is in `args[0]`. Replace that variable with a variable set by your app's runtime configuration or a path or UNC path as a string ([more](http://msdn.microsoft.com/en-us/library/system.io.fileinfo.aspx)).

Here's full [log4net docs](http://logging.apache.org/log4net/release/manual/configuration.html).

## Create log4net config file

Here's a sample log4net config file with 1 logger (the top-level [root logger](http://logging.apache.org/log4net/release/manual/introduction.html#hierarchy)) sending only to 1 appender. That [RemoteSyslogAppender](http://logging.apache.org/log4net/release/sdk/log4net.Appender.RemoteSyslogAppender.html) instance sends to Papertrail.

Logs from this app will be identified with the system's hostname and the app name `MyApp`, as defined in the config `Identity` option below. See [Tweak Identity and PatternLayout](#identity) for ways to control this.

In the example below, replace `logsN` and `XXXXX` with details from Papertrail's [Add Systems](https://papertrailapp.com/systems/setup) page.

```xml
<log4net>
  <appender name="PapertrailRemoteSyslogAppender" type="log4net.Appender.RemoteSyslogAppender">
    <facility value="Local6" />
    <identity value="%date{yyyy-MM-ddTHH:mm:ss.ffffffzzz} %P{log4net:HostName} MyApp" />
    <layout type="log4net.Layout.PatternLayout" value="%level - %message%newline" />
    <remoteAddress value="logsN.papertrailapp.com" />
    <remotePort value="XXXXX" />
  </appender>

  <root>
  <level value="DEBUG" />
  <appender-ref ref="PapertrailRemoteSyslogAppender" />
  </root>
</log4net>
```
<a name="identity"></a>

## Tweak Identity and PatternLayout

In the configuration above, the log message is defined with:

```xml
<identity value="%date{yyyy-MM-ddTHH:mm:ss.ffffffzzz} %P{log4net:HostName} MyApp" />
<layout type="log4net.Layout.PatternLayout" value="%level - %message%newline" />
```

This will generate a message like:

```
Mar 23 23:59:59 SYSTEM_NAME MyApp: INFO - the message
```

Both the identity and the format can be customized. The identity could include the name of your app and/or a fixed hostname.

The `PatternLayout` can include additional values, which are covered in log4net's  [PatternLayout documentation](https://logging.apache.org/log4net/release/sdk/log4net.Layout.PatternLayout.html). `file`, `logger`, and `%aspnet-request{key}` may be relevant to your application.

## Attributes

Consider using [ThreadContext](https://logging.apache.org/log4net/release/sdk/log4net.ThreadContext.html) to set arbitrary keys and values, such as a user ID or request ID. For example:

```csharp
ThreadContext.Properties["user"] = userName;
ThreadContext.Properties["request"] = "abc123";
```

# Use

Papertrail is just another log4net target, so no code changes should be needed for apps which already use log4net.

If you aren't yet using log4net, generate messages with:

```csharp
private static readonly ILog log = LogManager.GetLogger(typeof(MyApp));
log.Debug("Did it again!");
```

See [log4net docs](http://logging.apache.org/log4net/release/manual/configuration.html).
