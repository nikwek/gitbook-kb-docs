---
layout: default
category: configuration
title: Configuring centralized logging from PHP apps
sidebar_title: PHP
order: 150
---

# Configuring centralized logging from PHP apps

Papertrail can accept logs from any PHP app, including CakePHP, CodeIgniter,
Laravel, Symfony, Zend, and "plain" PHP.

If you aren't sure how to configure logging for your PHP app, start with the PHP manual coverage of [Error Handling](http://www.php.net/manual/en/book.errorfunc.php), especially [Error Handling Functions](http://www.php.net/manual/en/ref.errorfunc.php).

# Send log file with remote_syslog2

Configure your Web server or app to log to a file as usual, to log to a file as usual, then 
[transmit the log file](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix) to Papertrail using [remote_syslog2](https://github.com/papertrail/remote_syslog2).
This does not require any modifications to the app.

# Send events from PHP app

PHP apps can also send log messages directly to Papertrail. Here are a few ways to send syslog to Papertrail from PHP.

<p><a name="generic"></a></p>

## Generic PHP log sender

[Here is](https://gist.github.com/2220679) simple PHP code to construct and
transmit a UDP remote syslog log message.

As the code comments show, call that PHP function from your app (or from a
log handler) to transmit the message to Papertrail.

## Monolog

Consider [Monolog](https://github.com/Seldaek/monolog) and its [SyslogHandler](https://github.com/Seldaek/monolog/blob/master/doc/02-handlers-formatters-processors.md#log-to-files-and-syslog) or [SyslogUdpHandler](https://github.com/Seldaek/monolog/blob/master/doc/02-handlers-formatters-processors.md#log-specific-servers-and-networked-logging).
This works particularly well with Laravel and Laravel Forge, since Laravel
[already uses](https://laravel.com/docs/errors) Monolog.

Here is an example Monolog `SyslogUdpHandler` that sends to Papertrail:

```php
require 'vendor/autoload.php';

use Monolog\Logger;
use Monolog\Formatter\LineFormatter;
use Monolog\Handler\SyslogUdpHandler;

// Set the format
$output = "%channel%.%level_name%: %message%";
$formatter = new LineFormatter($output);

// Setup the logger
$logger = new Logger('my_logger');
$syslogHandler = new SyslogUdpHandler("logsN.papertrailapp.com", XXXXX);
$syslogHandler->setFormatter($formatter);
$logger->pushHandler($syslogHandler);

// Use the new logger
$logger->addInfo('Monolog test');
```

Change `logsN` and `XXXXX` to match your [Papertrail log destination](https://papertrailapp.com/account/destinations).

## Kohana

Users of the [Kohana](http://kohanaframework.org/) framework can use the [kohana-papertrail](https://github.com/dlucian/kohana-papertrail) module. Setup instructions can be found in the [README](https://github.com/dlucian/kohana-papertrail/blob/master/README.md).

# Logging via the Heroku add-on

Rather than logging directly to syslog, send everything to stderr and Heroku will forward it over to Papertrail. An example of how do this using Monolog, adapted from [this](https://devcenter.heroku.com/articles/getting-started-with-php#basic-logging) Heroku Dev Center article, is shown below.

```php
require('vendor/autoload.php');

use Monolog\Logger;
use Monolog\Formatter\LineFormatter;
use Monolog\Handler\StreamHandler;

// set the format
$output = "%message%";
$formatter = new LineFormatter($output);

// create a log channel to STDOUT
$log = new Logger('my_logger');
$streamHandler = new StreamHandler('php://stdout', Logger::WARNING);
$streamHandler->setFormatter($formatter);
$log->pushHandler($streamHandler);

// test messages
$log->addWarning("testing 1");
$log->addWarning("testing 2");
```