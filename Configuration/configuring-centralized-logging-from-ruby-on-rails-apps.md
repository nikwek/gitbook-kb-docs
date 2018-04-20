---
layout: default
category: configuration
title: Configuring centralized logging from Ruby on Rails apps
sidebar_title: Ruby on Rails
order: 160
---

# Configuring centralized logging from Ruby on Rails apps

To send Ruby on Rails request logs, either:

 * use Papertrail's tiny [remote_syslog2](#remote_syslog2) daemon to read an existing log file (like `production.log`), or
 * change Rails' environment config to use the [remote_syslog_logger](https://github.com/papertrail/remote_syslog_logger/) gem.

We recommend [remote_syslog2](#remote_syslog2) because it works for other text files (like nginx and MySQL), has no impact on the Rails app, and is easy to set up.

Also see [Controlling Verbosity](#verbosity).

<a name="remote_syslog2"></a>

# Send log file with remote_syslog2

REMOVED INCLUDE

<a name="remote_syslog_logger"></a>

# Send events with the `remote_syslog_logger` gem

## Install [Remote Syslog Logger](https://github.com/papertrail/remote_syslog_logger/)

The easiest way to install `remote_syslog_logger` is with Bundler. Add `remote_syslog_logger` to your `Gemfile`.

If you are not using a `Gemfile`, run:

```shell
$ gem install remote_syslog_logger
```

## Configure Rails environment

Change the environment configuration file to log via `remote_syslog_logger`.  This is almost always in `config/environment.rb` (to affect all environments) or `config/environments/<environment name>.rb`, such as `config/environments/production.rb` (to affect only a specific environment).  Add this line:

```ruby
config.logger = RemoteSyslogLogger.new('logsN.papertrailapp.com', XXXXX)
```

You can also specify a program name other than the default `rails`:

```ruby
config.logger = RemoteSyslogLogger.new('logsN.papertrailapp.com', XXXXX, :program => "rails-#{RAILS_ENV}")
```

where `logsN` and `XXXXX` are the name and port number shown under [log destinations](https://papertrailapp.com/account/destinations).

Alternatively, to point the logs to your local system, use `localhost` instead of `logsN.papertrailapp.com`, `514` for the port, and ensure that the system's syslog daemon is bound to `127.0.0.1`. A basic rsyslog config would consist of the following lines in `/etc/rsyslog.conf`:

```
$ModLoad imudp
$UDPServerRun 514
```

## Verify configuration

To send a test message, start `script/console` in an environment which has the syslog config above (for example, `RAILS_ENV=production script/console`).  Run:

```ruby
RAILS_DEFAULT_LOGGER.error "Salutations!"
```

The message should appear on the [system's message history](https://papertrailapp.com/dashboard) within 1 minute.

# Verbosity

For more information on improving the signal:noise ratio, see the dedicated help article [here](/kb/configuration/controlling-verbosity).

## Lograge

We recommend using [lograge](https://github.com/roidrage/lograge) in
lieu of Rails' standard logging. Add `lograge` to your `Gemfile` and
smile.

### Log user ID, customer ID, and more

Use lograge to include other attributes in log messages, like a user ID 
or request ID. The [README](https://github.com/roidrage/lograge) has more. 
Here's a simple example which captures 3 attributes:

```ruby
class ApplicationController < ActionController::Base
  before_filter :append_info_to_payload

  def append_info_to_payload(payload)
    super
    payload[:user_id] = current_user.try(:id)
    payload[:host] = request.host
    payload[:source_ip] = request.remote_ip
  end
end
```

The 3 attributes are logged in `production.rb`: with this block:

```ruby
config.lograge.custom_options = lambda do |event|
  event.payload
end
```

The `payload` hash populated during the request above is 
automatically available as `event.payload`. `payload` automatically
contains the params hash as `params`.

Here's another `production.rb` example which only logs the request 
params:

```ruby
config.lograge.custom_options = lambda do |event|
  params = event.payload[:params].reject do |k|
    ['controller', 'action'].include? k
  end

  { "params" => params }
end
```

## Troubleshooting

### Colors and/or ANSI character codes appear in my log messages

By default, Rails generates colorized log messages for non-production environments and monochromatic logs in production. Papertrail renders any ANSI color codes it receives (see [More colorful logging with ANSI color codes](http://blog.papertrailapp.com/post/49397820256/more-colorful-logging-with-ansi-color-codes)), so you can decide whether to enable this for any environment.

To enable or disable ANSI logging, change this option in your environment configuration file (such as `config/environment.rb` or `config/environments/staging.rb`). The example below disables colorized logging.

Rails >= 3.x:

```ruby
config.colorize_logging = false
```

Rails 2.x:

```ruby
config.active_record.colorize_logging = false
```

See: [http://guides.rubyonrails.org/configuring.html#rails-general-configuration](http://guides.rubyonrails.org/configuring.html#rails-general-configuration)

