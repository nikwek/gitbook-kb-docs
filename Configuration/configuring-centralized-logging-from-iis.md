---
layout: default
category: configuration
title: Configuring centralized logging from IIS
sidebar_title: IIS
order: 90
---

# Configuring centralized logging from IIS

IIS and Web apps running within it generate two types of log messages: access and error log files (from IIS itself), and app log messages (typically from a C# or ASP.NET app). Papertrail can accept logs from either or both.

To aggregate Windows event logs, see [Configuring remote syslog from Windows](/kb/configuration/configuring-remote-syslog-from-windows).

# Web app logs

For apps running in IIS, we recommend using either NLog or log4net as a logging library. For NLog, visit Papertrail's [NLog .NET logging](/kb/configuration/nlog-net-logging) setup page. For log4net, visit Papertrail's [log4net logging](/kb/configuration/log4net-logging) setup page.

# IIS access and error logs

For IIS access logs and error logs, aggregate them as standard text log files. See [Configuring remote syslog from Windows](/kb/configuration/configuring-remote-syslog-from-windows).

# Support

Please [ask](/) if we can help with configuration or troubleshooting.
