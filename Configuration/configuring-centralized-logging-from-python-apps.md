---
layout: default
category: configuration
title: Configuring centralized logging from Python apps
sidebar_title: Python
order: 155
---

# Configuring centralized logging from Python apps

Papertrail can accept logs from any Python app, including Django.

For accounts created via the Papertrail website, see methods A & B. For Heroku add-on accounts, see the [Heroku section](#logging-via-the-heroku-add-on) below.

# Send log file with remote_syslog2

Configure your Python app to log to a file, then [transmit the log file](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix) to Papertrail using [remote_syslog2](https://github.com/papertrail/remote_syslog2).
This typically does not require any modifications to the app.

# Send events using Python's SysLogHandler

Python can also send log messages directly to Papertrail with Python's [SysLogHandler](http://docs.python.org/library/logging.handlers.html#sysloghandler) logging handler.

Older minor versions of Python 2.7 and 3.2 may be subject to [this bug](http://bugs.python.org/issue14452). The fix is present in current 2.7 and 3.2 as well as all versions of 3.3 and later.

## Configuration

* For Django apps: [add a new logging handler](https://docs.djangoproject.com/en/dev/topics/logging/#an-example)
* For pure Python apps or those using frameworks other than Django: [instantiate](http://docs.python.org/howto/logging-cookbook.html#logging-cookbook) a `logging.handlers.SysLogHandler` instance

## Examples

### Django

Building on [this example](https://docs.djangoproject.com/en/dev/topics/logging/#an-example), add a new handler:

```python
'handlers': {
    'SysLog': {
        'level': 'DEBUG',
        'class': 'logging.handlers.SysLogHandler',
        'formatter': 'simple',
        'address': ('logsN.papertrailapp.com', XXXXX)
    },
    ...
}
```

Change the `address` argument to match [your Papertrail log destination](https://papertrailapp.com/systems/setup).

To set the sender and program name, modify the simple formatter like so:

```python
'formatters': {
    'simple': {
        'format': '%(asctime)s SENDER_NAME PROGRAM_NAME: %(message)s',
        'datefmt': '%Y-%m-%dT%H:%M:%S',
    },
    ...
}
```

Then add the new logging handler to at least one of the `loggers` in the config, for example:

```python
'loggers': {
    'django': {
        'handlers': ['file', 'SysLog'],
        'level': 'INFO',
        'propagate': True,
    },
    ...
}
```

### Generic Python app

```python
import logging
import socket
from logging.handlers import SysLogHandler


class ContextFilter(logging.Filter):
    hostname = socket.gethostname()

    def filter(self, record):
        record.hostname = ContextFilter.hostname
        return True

logger = logging.getLogger()
logger.setLevel(logging.INFO)

f = ContextFilter()
logger.addFilter(f)

syslog = SysLogHandler(address=('logsN.papertrailapp.com', XXXXX))
formatter = logging.Formatter('%(asctime)s %(hostname)s'
                              'YOUR_APP: %(message)s',
                              datefmt='%b %d %H:%M:%S')

syslog.setFormatter(formatter)
logger.addHandler(syslog)

logger.info("This is a message")
```

Change the `address` argument to match [your Papertrail account](https://papertrailapp.com/systems/setup).

## Format specifiers

These format specifiers have not been extensively tested but may be helpful as starting points, especially combined with the [logging format specifiers](http://docs.python.org/release/3.1.5/library/logging.html#basic-example).

A valid syslog message looks like this:

```
<22>Jan 2 23:34:45 hostname app_name[PID]: message
```

The timestamp can also be in other formats as shown in the code snippet above.

Here's an example format specifier for having Python's `SysLogHandler` generate syslog or syslog-like messages:

```python
logging.Formatter('%(asctime)s %(hostname)s APP: %(message)s', datefmt='%b %d %H:%M:%S')
```

The system hostname can be obtained with `socket.gethostname()`, by honoring the `HOSTNAME` environment variable with `os.getenv('HOSTNAME')`, or hardcoded as a static string literal. SysLogHandler automatically prefixes the message with a priority code, which is <22> in the example syslog message shown above.

Here's an alternative example that may be useful, depending on your environment:

```python
%(name)s[%(process)d]: %(levelname)s %(message)s
```

The [DatagramHandler](http://docs.python.org/release/2.5.2/lib/node415.html) may be used to similar effect.

<p><a name="example-multiline-log-messages"></a></p>

## Multiline log Messages

Syslog is a simple protocol that transports single lines of text. As a result, no encoding is provided for newlines. The message has to be split and separate packets have to go out over the wire.

For example, to log an exception, you'd use something like:

```python
tb = ""
try:
    a = 1/0
except:
    tb = traceback.format_exc()

lines = tb.split('\n')
for l in lines:
    logger.info(l)
```

# Logging via the Heroku add-on

Rather than logging directly to syslog, send everything to the console and Heroku will forward it over to Papertrail.

The following handler will allow logging to the console:

```python
import sys
LOGGING = {
    'handlers': {
        'console':{
            'level':'INFO',
            'class':'logging.StreamHandler',
            'strm': sys.stdout
        },
        ...
    }
}
```

Replace `strm` with `stream` if you're using python 2.7.
