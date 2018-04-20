---
layout: default
category: configuration
title: Java logback logging
sidebar_title: Java (logback)
order: 105
---

# Java logback logging

# Introduction

[logback](http://logback.qos.ch/) is "intended as a successor to the popular log4j project."

Papertrail supports aggregating messages from a native logback appender,
providing a live searchable console for your Java (JRE/JVM) app logs,
including Scala apps using frameworks like Lift and Play.

Papertrail can receive logback messages using
[logback-syslog4j](https://github.com/papertrail/logback-syslog4j)
(`com.papertrailapp.logback.Syslog4jAppender`) or the older original
logback `ch.qos.logback.classic.net.SyslogAppender`. As described below,
we recommend [logback-syslog4j](https://github.com/papertrail/logback-syslog4j).

# Install logback

Install [logback](http://logback.qos.ch/setup.html) in your app or J2EE
servlet container. This is basically adding the jars to your classpath,
creating a [logback.xml](http://logback.qos.ch/manual/configuration.html), and
loading and invoking the classes to output messages (such as
`logger.debug("Success!");`).

# Choose appender

logback outputs logs to one or more appenders (such as file, console, or
syslog). Papertrail recommends the [logback-syslog4j](https://github.com/papertrail/logback-syslog4j)
appender (`com.papertrailapp.logback.Syslog4jAppender`) because it
includes TCP, TCP with TLS, and Google App Engine support, as well as
relying on [syslog4j](http://syslog4j.org/) for message transmission.

If adding an appender to `pom.xml` is not possible, or you prefer the
standard logback [SyslogAppender](http://logback.qos.ch/manual/appenders.html#SyslogAppender),
this appender can also can send to Papertrail.

## Syslog4jAppender (recommended)

Visit [logback-syslog4j README](https://github.com/papertrail/logback-syslog4j)
and follow the step-by-step instructions.

In the `logback.xml` example in the README, replace
`host` and `port` with the details from Papertrail's
[Add Systems](https://papertrailapp.com/systems/setup) page. Replace
`ident` with a name of your app, or other attribute that Papertrail
should use to identify the sending app.

### Customize sender name and identifier

To change the appearance of the sender name and program/calling method
name in Papertrail:

* add `<sendLocalTimestamp>false</sendLocalTimestamp>` & `<sendLocalName>false</sendLocalName>` to `<syslogConfig>`
* remove the `<ident>` tag
* replace the layout pattern under under `Syslog4jAppender` with the following:

```xml
<pattern>%d{MMM dd HH:mm:ss} HOSTNAME APPNAME: %-5level %logger{35}: %m%n%xEx</pattern>
```

`HOSTNAME` will appear as the sender name in Papertrail, and `APPNAME` will appear as the program.

# SyslogAppender

Edit `logback.xml` and add this within the `<appenders>` block.

In the sample below, replace `logsN` and `XXXXX` with the details from Papertrail's [Add Systems](https://papertrailapp.com/systems/setup)
page. Replace `my-app` with a name of your app, or other attribute that
Papertrail should use to identify this sender.

```xml
<appender name="PAPERTRAIL" class="ch.qos.logback.classic.net.SyslogAppender">
  <syslogHost>logsN.papertrailapp.com</syslogHost>
  <port>XXXXX</port>
  <facility>USER</facility>
  <suffixPattern>my-app: %logger %msg</suffixPattern>  
</appender>
```

## Enable SyslogAppender

Outside of the appenders section (in the `<root>` block), add this to enable the syslog appender you just created:

```xml
<appender-ref ref="PAPERTRAIL" />
```

An example might look like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{5} -
%msg%n</pattern>
    </encoder>
  </appender>

  <root level="DEBUG">
    <appender-ref ref="STDOUT" />
    <appender-ref ref="PAPERTRAIL" />
  </root>
</configuration>
```

# Formatting

An alternative `suffixPattern` that puts the `%thread` variable in the position that's used for clickable [context](/kb/how-it-works/event-viewer#context) (instead of `my-app` above):

```xml
<suffixPattern>%thread: %-5level %logger{36} - %msg%n</suffixPattern>
```

# Troubleshooting

## I'm using the Spark framework and no logs are appearing in Papertrail

Spark and logback rely on SLF4J, so to avoid including it twice it must be excluded as a dependency.

## Enable logback debug mode

logback supports a [debug mode](http://jira.qos.ch/browse/LOGBACK-527) for its own operations. To enable it, change the outer `<configuration>` tag to `<configuration debug="true">`.

# More

* [Papertrail Logging with Lift and AWS Beanstalk](http://richard.dallaway.com/papertrail-lift-beanstalk.html)
