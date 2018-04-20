---
layout: default
category: configuration
title: Java log4j logging
sidebar_title: Java (log4j)
order: 95
---

# Java log4j logging

# Introduction

[log4j](http://logging.apache.org/log4j/) is "a popular logging package written in Java. One of its distinctive features is the notion of inheritance in loggers. Using a logger hierarchy it is possible to control which log statements are output at arbitrary granularity."

Before implementing log4j on a new app, seriously consider using:

* [logback](http://logback.qos.ch/), intended as a successor to log4j
* [SLF4J](http://www.slf4j.org/), a logging abstraction layer which supports multiple pluggable backends (including log4j, logback, and java.util.Logging)

Papertrail supports aggregating messages from a native log4j 1.x or log4j 2 appender, providing a live searchable console for your Java (JRE/JVM) app logs. 

# log4j 1.x

## Installation

Install [log4j](http://logging.apache.org/log4j/1.2/download.html) in your app or J2EE servlet container. This is basically adding the log4j jar to your classpath, creating a [config file](http://wiki.apache.org/logging-log4j/Log4jXmlFormat), and loading and invoking the classes to [output messages](https://www.google.com/search?q=log4j+getRootLogger).

Here are more specifics for using log4j in:

* [Tomcat](http://tomcat.apache.org/tomcat-6.0-doc/logging.html)
* [JBoss](http://docs.jboss.org/process-guide/en/html/logging.html)

log4j includes [SyslogAppender](http://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/net/SyslogAppender.html) in the standard distribution. log4j 1.2.14 and newer support user-specified destination ports, which makes Papertrail configuration easier. The steps below assume 1.2.14 or newer. log4j's SyslogAppender uses [UDP syslog](/kb/configuration/troubleshooting-remote-syslog-reachability/#udp).

## Setup SyslogAppender for Papertrail

Edit `log4j.xml` and add the sample below within the `<log4j:configuration>` block. Replace `logsN` and `XXXXX` with the details from Papertrail's [Add Systems](https://papertrailapp.com/systems/setup) page.

```xml
<appender name="SYSLOG" class="org.apache.log4j.net.SyslogAppender">
  <errorHandler/>
  <param name="Facility" value="LOCAL7"/>
  <param name="FacilityPrinting" value="false"/>
  <param name="Header" value="true"/>
  <param name="SyslogHost" value="logsN.papertrailapp.com:XXXXX"/>
  <param name="ConversionPattern" value="%p: %c{2} %x %m %n"/>
</appender>
```

<div class="alert alert-info" role="alert">
  <div class="fa fa-info-circle alert-icon"></div>
  <div class="alert-message">Where the config snippet above has <code>&lt;errorHandler/&gt;</code>, you may need to provide a <code>class</code> attribute that is your app server's <a href="http://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/spi/ErrorHandler.html">ErrorHandler</a> implementation. </div>
</div>

For example, for Tomcat:

```xml
<errorHandler class="org.apache.log4j.helpers.OnlyOnceErrorHandler" />
```

or for JBoss:

```xml
<errorHandler class="org.jboss.logging.util.OnlyOnceErrorHandler" />
```

Alternatively, to use `log4j.properties`, add the following (replacing `logsN` and `XXXXX` as explained above):

```
log4j.rootLogger=INFO, syslog

log4j.appender.syslog=org.apache.log4j.net.SyslogAppender
log4j.appender.syslog.Facility=LOCAL7
log4j.appender.syslog.FacilityPrinting=false
log4j.appender.syslog.Header=true
log4j.appender.syslog.SyslogHost=logsN.papertrailapp.com:XXXXX
log4j.appender.syslog.layout=org.apache.log4j.PatternLayout
log4j.appender.syslog.layout.ConversionPattern=%p: (%F:%L) %x %m %n
```

## Enable appender

Outside of the appenders section (in the `<root>` block), add this to enable the syslog appender you just created:

```xml
<appender-ref ref="SYSLOG" />
```

## Change sender identifier (optional)

With the configuration option `Header` above, `SyslogAppender` automatically prepends the timestamp (in `MMM dd HH:mm:ss` format) and the system hostname, which Papertrail uses as the sender identifier. In most cases, this is the best sender identifier.

To use a different sender identifier (such as the name of the app or an arbitrary string), disable the `Header` option and include those within the `ConversionPattern`. This configuration:

```xml
<param name="Header" value="true"/>
<param name="ConversionPattern" value="%p: %c{2} %x %m %n"/>
```

Is functionally equivalent to this one, except that this configuration is manually outputting the date and sender ID `my-app` instead of relying on log4j's `Header` option:

```xml
<param name="Header" value="false"/>
<param name="ConversionPattern" value="%d{MMM dd HH:mm:ss} my-app %p: %c{2} %x %m %n"/>
```

When editing the `ConversionPattern`, retain the `%d{MMM dd HH:mm:ss} my-app ` at the start.

## Edit format (optional)

Finally, you can optionally edit the message format using standard log4j output formatting parameters. To do so, include a `<layout>` block within `<appender>`. Here's an extremely verbose example:

```xml
<layout>
  <param name="ConversionPattern" value="%t %5r %-5p %c{2} [%x] %m %n"/>
</layout>
```

For more details about the `ConversionPattern` you can refer to the documentation for [`PatternLayout`](http://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/PatternLayout.html).

## Use

Papertrail is just another log4j target, so no code changes should be needed.

If you aren't yet using log4j, generate messages with a snippet like this. From the [log4j manual](http://logging.apache.org/log4j/1.2/manual.html):

```java
// get a logger instance named com.foo.Bar
Logger barlogger = Logger.getLogger("com.foo.Bar");
barlogger.warn("Low fuel level.");
barlogger.debug("Starting search for nearest gas station.");
```

# log4j 2

## Installation

See [installation](https://logging.apache.org/log4j/2.0/download.html). Ensure that the API and core .jar files are present in the application's classpath.

## Setup SYSLOG-TCP appender for Papertrail

<div class="alert alert-warning" role="alert">
  <div class="fa fa-exclamation-triangle alert-icon"></div>
  <div class="alert-message">Papertrail does not enable plaintext TCP by default. To enable it, visit <a href="https://papertrailapp.com/account/destinations">Log Destinations</a>, click <strong>Edit Settings</strong>, and enable plaintext TCP logging.</div>
</div>

This does not capture stack traces.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" name="MyApp" packages="">
  <Appenders>
    <Syslog name="SYSLOG-TCP" host="logsN.papertrailapp.com" port="XXXXX"
      protocol="TCP" appName="MyApp" mdcId="mdc"
      facility="LOCAL0" enterpriseNumber="18060" newLine="true"
      format="RFC5424" ignoreExceptions="false" exceptionPattern="%throwable{full}">
    </Syslog>
  </Appenders>
  <Loggers>
    <Root level="debug">
      <AppenderRef ref="SYSLOG-TCP"/>
    </Root>
  </Loggers>
</Configuration>
```

In the sample above, replace `logsN` and `XXXXX` with the details from Papertrail's [Add Systems](https://papertrailapp.com/systems/setup) page.