---
layout: default
category: configuration
title: Unicorn
order: 165
---

# Unicorn

[Unicorn](http://unicorn.bogomips.org/) is a high-performance forking Web server that is often used for serving Ruby on Rails applications.

Many times, its logging configuration leads implementors to re-instantiate Rails' `logger` object, and in the process, to unintentionally log at `DEBUG` log level (with SQL statements, cache hits, and other events that are often not necessary).

Here's how to change the log level of a Rails app served by Unicorn.

# Problem

Unicorn doesn't load `Rails::Rack::LogTailer`, which is what outputs most Rails logs to `STDOUT`. This isn't an inherent problem, it's simply a different approach to logging.

Often this leads someone to instantiate a new `Logger` in the environment-specific Rails initializer which in the process sets the log level to `DEBUG` verbosity.

# Solution

Here's how to replicate similar behavior as exists with most other Rails Web servers.

To output logs to `STDOUT` and define the log level, edit `config/environments/<environment>.rb`, such as `production.rb`. In the `MyApp::Configuration.configure .. end` block, add these 3 lines:

```ruby
config.logger = Logger.new(STDOUT)
config.logger.level = Logger.const_get('INFO')
config.log_level = :info
```

These create a new `Logger` class that outputs to `STDOUT`, then
changes the logging level in that newly-instantiated `logger` (to an
integer, as returned by `const_get`).

Finally, it sets Rails' `log_level` to the equivalent symbol so that any
other callers, such as Heroku's `rails_12factor` gem, also see the
updated level (example: [rails_stdout_logging](https://github.com/heroku/rails_stdout_logging/blob/master/lib/rails_stdout_logging/rails.rb#L11)).

Comment out any existing assignments of `config.logger` or `config.log_level` in the file.

# Advanced options

## Use existing logger

The example above creates a new logger (to `STDOUT`) and sets its log
level. If you want to leave the log destination as-is and only change
its log level, use this instead of the lines above:

```ruby
config.logger.level = Logger.const_get('INFO')
config.log_level = :info
```

Instead of redefining `config.logger` and setting its `level`, this
simply sets `level` on the existing `config.logger`.

## Read log level from env variable

To set the log level as an environment variable (like when using Heroku
config variables), use this instead of the lines above:

```ruby
config.logger = Logger.new(STDOUT)
config.logger.level = Logger.const_get(ENV['LOG_LEVEL'] ? ENV['LOG_LEVEL'].upcase : 'INFO')
config.log_level = (ENV['LOG_LEVEL'] ? ENV['LOG_LEVEL'].downcase : 'info').to_sym
```

This will use the `LOG_LEVEL` environmental variable when it is set and
`INFO` when it is not set.

# Unicorn on Heroku

As of this writing, A Rails 3.1.3 app using Unicorn 4.1.1 on Heroku (cedar) outputs few or no app logs in its initial configuration. Read more [here](https://github.com/ryanb/cancan/issues/511#issuecomment-3643266) or [here](http://groups.google.com/group/heroku/browse_thread/thread/119c52ba08d173b4).

When a Rails app is deployed, Heroku injects
[rails_log_stdout](https://github.com/ddollar/rails_log_stdout), which
redefines Rails' `Logger` so all output goes to `STDOUT`. This doesn't
work as expected with Unicorn, so only rack output is logged.

To re-enable logging to `STDOUT` like Heroku expects, use the example in
[Read log level from env variable](#read-log-level-from-env-variable)
above.

Use `heroku config:add` to set the desired log level of the Heroku app.
For example:

```shell
$ heroku config:add LOG_LEVEL="warn"
```

The app will restart immediately at the new log level.

## Disable `rails_log_stdout` (optional)

Optionally, you can tell Heroku not to inject [rails_log_stdout](https://github.com/ddollar/rails_log_stdout) since it's no longer relevant with Unicorn.

To do so, create an empty `vendor/plugins/rails_log_stdout/` directory containing a placeholder file and commit it to your git repo:

```shell
$ mkdir -p vendor/plugins/rails_log_stdout
$ touch vendor/plugins/rails_log_stdout/keep_me
$ git add vendor/plugins/rails_log_stdout/keep_me
$ git commit -m "Placeholder file to disable rails_log_stdout" vendor/plugins/rails_log_stdout/keep_me
```