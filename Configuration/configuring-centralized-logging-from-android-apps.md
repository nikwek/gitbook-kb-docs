---
layout: default
category: configuration
title: Configuring centralized logging from Android apps
sidebar_title: Android
order: 55
---

# Configuring centralized logging from Android apps

Papertrail can accept logs from any Android application using either of the following methods.

# logback-syslog4j

* Add the following to the app's Gradle dependencies list:

```gradle
compile 'org.slf4j:slf4j-api:1.7.13' 
compile 'com.github.tony19:logback-android-core:1.1.1-4' 
compile 'com.github.tony19:logback-android-classic:1.1.1-4' 
compile ('com.papertrailapp:logback-syslog4j:1.0.0'){ 
  exclude group: 'ch.qos.logback' 
}
```
 
* Use this configuration file, replacing `logsN` and `XXXXX` with the details shown
under [Log Destinations](https://papertrailapp.com/account/destinations) in Papertrail.
`YOUR_APP` will appear as the program name in the event viewer; replace this with your desired app name.

```xml
<configuration> 
  <appender name="syslog-tls" class="com.papertrailapp.logback.Syslog4jAppender"> 
    <layout class="ch.qos.logback.classic.PatternLayout">
      <pattern>%d{MMM dd HH:mm:ss} Android YOUR_APP: %-5level %logger{35} %m%n</pattern>
    </layout>

    <syslogConfig class="org.productivity.java.syslog4j.impl.net.tcp.ssl.SSLTCPNetSyslogConfig"> 
      <host>logsN.papertrailapp.com</host> 
      <port>XXXXX</port> 
      <sendLocalName>false</sendLocalName>
      <sendLocalTimestamp>false</sendLocalTimestamp> 
      <maxMessageLength>128000</maxMessageLength> 
    </syslogConfig> 
  </appender>

  <appender name="async" class="ch.qos.logback.classic.AsyncAppender"> 
    <appender-ref ref="syslog-tls" /> 
  </appender>

  <root level="INFO"> 
    <appender-ref ref="async" /> 
  </root> 
</configuration>
```

# logback-android

Use [logback-android](http://tony19.github.io/logback-android/) and then follow
Papertrail's typical [logback setup](/kb/configuration/java-logback-logging/#alternative-older-syslogappender).

The article [Logging from iOS and macOS apps](/kb/configuration/configuring-centralized-logging-from-ios-or-os-x-apps/)
has a few tips for making the most of Papertrail for logs from mobile devices.

# Other methods

If neither `logback-syslog4j` or `logback-android` are suitable for your app:

* Look at [Logging from iOS and macOS apps](../configuring-centralized-logging-from-ios-or-os-x-apps) for additional language-independent logging methods, like simply sending UDP syslog packets to Papertrail.
* Let us know. We can usually come up with something and we enjoy the chance.
