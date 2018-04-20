---
layout: default
category: how-it-works
title: Alerts
order: 10
---

# Alerts

Here's how to receive certain log messages to email or send them to services like Slack, Librato, PagerDuty, Campfire, and your own custom HTTP webhooks.

# Introduction

Papertrail can notify external services when new log messages match important searches. These notifications can happen every minute (like for a monitoring system), every hour, or every day, and only occur in periods when Papertrail receives new matches for a given search.

For background, see the search alert announcement [blog post](http://blog.papertrailapp.com/post/8189374546/announcing-papertrail-search-alerts).

Interested in alerting when something _doesn't_ happen, like a cron job that doesn't run? Check out [Inactivity alerts](#inactivity-alerts).

# Create alert

To create an alert, save a search, then attach an alert.

1. **Save a search.** In [Events](https://papertrailapp.com/events), search for the logs that Papertrail should alert on. Once the matches are refined, click **Save Search**: ![save_search_example.jpg](/assets/images/save_search_example_normal.jpg){:width="600" height="40"}
2. **Attach an alert.** Give the new saved search a name, and click **Save & Setup an Alert**: ![save_and_setup_an_alert-1.jpg](/assets/images/save_and_setup_an_alert-1_normal.jpg){:width="498" height="280"}

## Existing search

To add an alert to an existing saved search, visit the [Dashboard](https://papertrailapp.com/dashboard), click the pencil icon on the saved search, then click **New Alert**.

# Configuration

After following the instructions above, the last step is to choose the alert service and configure it. The _Choose a Service_ page looks like this:

![manage_alerts.png](/assets/images/manage_alerts.png)

# Supported services

Papertrail can notify:

* [Campfire](http://campfirenow.com/): send a chat message to a Campfire room. It contains the logs and a link. [More](/kb/integrations/campfire).
* [Datadog](https://www.datadoghq.com/): graph the number of occurrences as Datadog metrics. [More](/kb/integrations/datadog).
* Emails: send an email to a set of addresses of your choosing.
* [GeckoBoard](http://www.geckoboard.com/): update a custom "number" widget with the count of matches. [More](/kb/integrations/geckoboard).
* [HipChat](http://hipchat.com/): send a chat message to a HipChat room. It contains the logs and a link. [More](/kb/integrations/hipchat).
* [HostedGraphite](http://hostedgraphite.com/): graph the number of occurrences as HostedGraphite metrics. [More](/kb/integrations/hostedgraphite).
* [Librato](http://metrics.librato.com/): graph the number of occurrences as Librato metrics. [More](/kb/integrations/librato-metrics).
* [PagerDuty](http://pagerduty.com/): invoke an alert escalation policy, such as to generate push notifications and text messages. [More](/kb/integrations/pagerduty).
* [Pushover](https://pushover.net/): Send iOS or Android push notification.
* [StatHat](http://stathat.com/): graph the number of occurrences as StatHat metrics. [More](/kb/integrations/stathat).
* [Slack](https://slack.com/): send a chat message to a Slack channel. It contains the logs and a link. [More](/kb/integrations/slack).
* [SNS](http://aws.amazon.com/sns/): deliver to Amazon Simple Notification Service. [More](/kb/integrations/amazon-sns).
* [VictorOps](https://victorops.com/): Notify staff when critical events occur.
* Webhook: notify a HTTP URL of your choosing. See [Web hooks](/kb/how-it-works/web-hooks).
* [Zapier](http://www.zapier.com): Feed logs into many other services, like Google Sheets, Flowdock, and Twitter. [More](/kb/integrations/zapier).

In addition, Papertrail has non-alert integrations with services like
[Honeybadger](/kb/integrations/honeybadger),
[New Relic](/kb/integrations/new-relic),
[OpsGenie](/kb/integrations/opsgenie),
and an macOS Dashboard [widget](https://papertrailapp.com/info/widget).
Expand the "Integration" sidebar menu section for additional
integrations.

Don't see one you need? Just ask or [implement](https://github.com/papertrail/papertrail-services).

# Schedule

Alerts are processed every minute, every hour at about the minute that the alert was created, and every day at about 5 AM in the timezone which the alert's messages will be timestamped to. Examples:

* A daily alert with a timezone of Eastern Time will be invoked at about 5 AM Eastern Time.
* An hourly alert created at 5:43 will be invoked at about 6:43, 7:43, and so forth.

NOTE: Alerts are not guaranteed to start and end on the same second. The impact of this is that 2 alerts with the same interval, that are associated with the same saved search, may not be triggered at the same time.

For example, say that one matching log line arrives at 09:01:10. A per minute alert that considered log lines from 09:00:09 - 09:01:09, wouldn't find any matches. Another alert that had a start time of 1 second later would fire as expected.

## Daily alert at a specific time

Daily alerts fire at approximately 5 AM in the timezone associated with the alert. The latter can be used to change when the alert executes in local time.

For example, if a daily alert needs to run at 8 AM Pacific standard time (GMT-8), it could be implemented by setting the alert's timezone to Samoa (GMT-11).

Note that timestamps inside alert notifications will also reflect the alert's timezone.

# Minimum threshold

Optionally, choose how many matching events must occur in the time interval chosen. For example, if a search alert runs every 10 minutes and should only be invoked when 5 or more events occur during that window, enter `5`. The default and most common value is `1`, which means to invoke the alert any time at least one matching message has occurred.

### Log velocity notifications

This can also serve as finer-grained notification when [log velocity](https://papertrailapp.com/account#log-rate) changes significantly. To use alerts as fine-grained velocity notifications, create a search that matches all logs (such as `" "`), then an alert with a relatively high minimum threshold. For example, a 1-minute alert interval could have a minimum of 24,000 in a minute, or an average of 400 per second.

Note that thresholds higher than 25,000 per minute (just over 400 per second) can't be tracked this way, because of the 25,000 maximum on search results per invocation.

# Maximum

For each alert invocation, Papertrail will truncate matching log messages to at most:

* 25,000 log messages, and
* 10 megabytes of JSON-encoded log messages (including related fields)

# Log velocity notifications

An alert can also serve as a finer-grained notification when [log velocity](https://papertrailapp.com/account#log-rate) changes significantly. To use alerts as fine-grained velocity notifications, create a search which matches all logs (such as `" "`), then an alert with a relatively high minimum threshold. For example, a 1-minute alert interval could have a minimum of 30,000 in a minute, or an average of 500 per second.

Note that thresholds higher than 120,000 per minute (that is, 5,000/second) may yield less predictable alert invocations due to variations in Papertrail's processing rate. The alerts will likely serve the purpose as velocity notifications, but the specific rate should not be considered authoritative.

# Custom services

To create your own alert service or extend the set of services that
Papertrail supports, visit [Web hooks](/kb/how-it-works/web-hooks).

# Inactivity alerts

The above instructions cover the setup for alerting on matching events, but what about when no events were sent because something didn't happen? When a cron job, backup, or other recurring job doesn't run, it's not easy to notice the absence of an expected message. But Papertrail can do the noticing for you.

## Set up an inactivity alert

Follow the usual process to [create a search and set up an alert](#create-alert). Then, from the create/edit alert form, choose **Trigger when ... no new events match**.

![Inactivity alerts](/assets/images/inactivity-alerts.png){: width="700"}

Once saved, the alert will send notifications when there are no matching events within the chosen time period. Use this for:

- cron jobs
- background jobs that should run nearly all the time, like system monitors/pollers and database or offsite backups
- lower-frequency scheduled jobs, like nightly billing

## Example

If you have cron jobs, backup jobs, or other recurring or scheduled jobs, they probably generate logs already. Here's how to have Papertrail tell you when they don't run, or run but don't complete successfully:

1. Search for the message emitted when a cron job finishes successfully (example: [cron](https://papertrailapp.com/events?q=cron)).
2. Click **Save Search**.
3. [Attach an alert](#create-alert), like to notify a Slack channel or send an email.

### No logs? No problem.

Occasionally, a recurring job doesn't generate log messages on its own. For those, use the shell `&&` operator and `logger` to generate a post-success log message. For example, `./run-billing-job && logger "billing succeeded"` will send the message billing succeeded to syslog if and only if `run-billing-job` finishes with a successful exit code. Use `"billing succeeded"` as the Papertrail search.

## Questions?

Give inactivity alerts a try and if you have questions or feedback, [let us know](mailto:support@papertrailapp.com).
