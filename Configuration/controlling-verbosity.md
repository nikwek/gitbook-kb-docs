---
layout: default
category: configuration
title: Controlling verbosity
order: 40
---

# Controlling Verbosity

# Rails

By default, Rails logs are extremely verbose. For example, every partial rendered generates a log message, even in production. Most of the verbosity is rarely useful, and often it's simply noise that has no value at all.

Here are a few recommendations which improve the signal:noise. We're happy to help implement or tweak any of these.

## Improve formatting

* **Log 1 line per request**: use a gem like [lograge](https://github.com/roidrage/lograge) or a Rails initializer like [this one](https://gist.github.com/f280eba1f4e0a412d800b401aecd084e) to generate far denser log messages, usually with no loss in information detail. For example:

```
1.2.3.4 GET /path 200 OK BlahController#action HTML 938.2 (DB 11.8, View 719.7) {params} {optional params set in flash[:log]}
```

See:

* standalone logging [initializer](https://gist.github.com/3310392)

## Reduce noise

To filter noise, you can filter log messages at the sender or on Papertrail ([log filtering](/kb/how-it-works/log-filtering)). Some ideas what to filter:

* **Static asset requests**: Disable logging requests for static assets (asset pipeline). In Rails >= 4.2.7, [use `config.assets.quiet`](https://github.com/rails/sprockets-rails#initializer-options). In older Rails versions or Rails apps using a non-Sprockets asset pipeline, try the [quiet_assets gem](https://github.com/evrone/quiet_assets) or remote_syslog's [exclude_patterns](https://github.com/papertrail/remote_syslog2#excluding-lines-matching-a-pattern).
* **Unnecessary actions**: disable logging requests for certain actions, like with the [ciunas gem](https://github.com/mmrwoods/ciunas) or [silencer](https://github.com/stve/silencer). If the vast majority of your requests are for a single action or group of actions (for example, a monitoring action or a very simple GET API request), and you don't use those messages, the log messages are noise that only distract from real visibility. Please note that `ciunas` is not thread-safe (and `silencer` is only thread-safe in Rails >= 4.2.6).

See:

* [quiet_assets gem](https://github.com/evrone/quiet_assets)
* remote_syslog's [exclude_patterns](https://github.com/papertrail/remote_syslog2#excluding-lines-matching-a-pattern)
* [log filtering](/kb/how-it-works/log-filtering) within Papertrail
* [ciunas gem](https://github.com/thickpaddy/ciunas)

# Control verbosity

* **Confirm existing verbosity**: make sure your app is not unintentionally logging at `DEBUG` level. This is much more common than it sounds. `DEBUG` is the default when a new `Logger` class is instantiated and the level is not changed. Often a new `Logger` is instantiated during deployment. Here's more on Heroku [log levels](http://stackoverflow.com/questions/8098429/heroku-logging-not-working). Also, Rails 5 changes the default production log level from `INFO` to `DEBUG`.
* **Decrease verbosity**: move to `INFO` or `WARN` log level in production. By default, Rails 4.2 and prior use `INFO` log level in production and `DEBUG` in other environments. Rails 5 and after use `DEBUG` everywhere. For almost all apps, `INFO` is more appropriate in production, and for many apps, `WARN` is. Even in situations where one actually wants extreme visibility, Rails' default `DEBUG` output is probably not the right choice; not many administrators need a list of every partial rendered in a given view.
