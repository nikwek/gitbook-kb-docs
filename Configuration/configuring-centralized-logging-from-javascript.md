---
layout: default
category: configuration
title: Configuring centralized logging from JavaScript
sidebar_title: JavaScript
order: 110
---

# Configuring centralized logging from JavaScript

Logging messages from a client browser, like from Backbone, Angular, Ember, or simple `console.log`-style statements? Papertrail has two uniquely flexible ways for aggregating JavaScript log messages. Use the one that works best for your environment.

Logging from a Node.js server rather than a client-side JavaScript app? See [Node.js](/kb/configuration/configuring-centralized-logging-from-nodejs-apps).

<a name="1-line-bridge-app"></a>

# Create a logging "bridge" app

**Consider this solution regardless of whether you use Heroku**. It's elegant and powerful.

Papertrail integrates with [Heroku](https://heroku.com/) by automatically collecting all output from a Heroku app and from the Heroku platform. Heroku automatically logs each request.

* Create a 1-line app on Heroku that does nothing except return HTTP 200. Use any [language](https://devcenter.heroku.com/categories/language-support) that Heroku supports. 
* Make a GET request to the bridge app from the client browser with query params containing the information to log. 

This is a ready-made log bridge with almost zero effort.

A common way to make supplementary requests as GETs is with an inlined 1x1 image, like this:

```html
<img src="https://your-app-name.herokuapp.com/log?event=somethingHappened" width=1 height=1 style="display: none" />
```

Heroku's service is free at low volume, so the setup doesn't add any cost. The JavaScript-to-syslog bridge can log to an existing standard Papertrail account or to an add-on account set up just for this. See [Heroku setup](/kb/hosting-services/heroku).

## What if I want to do more?

The no-op app is a great basic solution, but there are lots of other things you can do, including augment or format the logs and use asynchronous methods to send data back:

* Add information not known by the browser, such as the user's Internet-facing IP
* Decide how and whether to handle formatting and authentication
* Output post-processed messages rather than raw query parameters
* Send logs however works best for you: as standard async HTTP requests, as synchronous HTTP requests, with a batch of logs that your Backbone app queues during a request, or a combination
* Log anything, from strings to hashes to JSON

For example, Vimeo's [Tattletale.js](https://github.com/vimeo/tattletale.js) logging library can send data to Papertrail:

```js
var tattletale = new Tattletale('https://probat-arabica-29.herokuapp.com/log');
tattletale.log('“My name is Ozymandias, king of kings:');
tattletale.log('Look on my works, ye Mighty, and despair!”');
tattletale.log(42);
tattletale.send();
```

Since Tattletale uses async XHR, a bridge app accepting data from Tattletale requires [Cross-origin resource sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS) (CORS) to be configured on the app -- no longer a no-op, but an easy step in many web frameworks. An example Sinatra app supporting CORS and deployable to Heroku is available [here](https://github.com/papertrail/js-log-bridge-app).

# Standalone app as bridge

The example above uses Heroku's add-on log integration to send logs to Papertrail, but Heroku isn't required. You can set up an app and run it anywhere. Since Papertrail accepts [syslog](https://tools.ietf.org/html/rfc5424) (a simple text format for sending logs over the wire), the "bridge" app is a few lines in any common language.

Here's an example using Papertrail's Ruby [remote_syslog_logger](https://github.com/papertrail/remote_syslog_logger) gem to generate the syslog message. Your JavaScript app hits this bridge app and the bridge outputs the log message to Papertrail.

## Example app

Using [Sinatra](http://www.sinatrarb.com), implementing a route to process log messages would look something like this:

```ruby
post '/message'
  logger = RemoteSyslogLogger.new('logs.papertrailapp.com', 11111, :local_hostname => 'an-existing-sender', :program => 'js-bridge')
  logger.info("Whatever you want, like: #{params[:order_id]} from #{request.remote_ip}")
```

Using a POST route as shown in this example would require CORS support (see [example app](https://github.com/papertrail/js-log-bridge-app)).

# Using an existing app

If the client-side app is associated with an existing server-side app, adding a route to that app to process client log messages (similar to the above example) may be easier than setting up a new app, especially if the server app already logs to Papertrail.

# Still not sure?

If we can help set up or deploy any of this, or if you're using another language and would like an example, please [contact us](/).
