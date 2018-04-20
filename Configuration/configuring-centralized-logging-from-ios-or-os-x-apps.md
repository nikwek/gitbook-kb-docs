---
layout: default
category: configuration
title: Configuring centralized logging from iOS or macOS apps
sidebar_title: Apple iOS and macOS
order: 60
---

# Configuring centralized logging from iOS or macOS apps

Papertrail can accept logs from any iOS or macOS application using either of the following methods. For both methods, your code determines the name Papertrail uses to identify each log sender. After setup, see [choosing sender name](#choosing-sender-name) below.

# PaperTrailLumberjack

[PaperTrailLumberjack](https://bitbucket.org/rmonkey/papertraillumberjack/) is a [CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack) logger that supports logging via messages sent over UDP and TCP. It can be easily integrated into your Cocoa project via [CocoaPods](https://cocoapods.org) or [Carthage](https://github.com/Carthage/Carthage).

## Installation

### CocoaPods

CocoaPods is a dependency manager for Cocoa Projects. To install PaperTrailLumberjack, add the following lines (as appropriate) to your PodFile:  

```objc
// Objective-C
use_frameworks!

target "YourTargetName" do
  pod "PaperTrailLumberjack"
end
```

```swift
/// Swift
use_frameworks!

target "YourTargetName" do
  pod "PaperTrailLumberjack/Swift"
end
```

...and import the `PaperTrailLumberJack` header:

```objc
// Objective-C
import <PaperTrailLumberjack/PaperTrailLumberjack.h>
```

```swift
/// Swift
import PaperTrailLumberjack
```

### Carthage

Carthage is a lightweight dependency manager for Cocoa applications. Detailed instructions on using Carthage are available [here](https://github.com/Carthage/Carthage). To import PaperTrailLumberjack, add the following line to your Cartfile:
   
```shell
git "https://bitbucket.org/rmonkey/papertraillumberjack.git"
```

## Usage

PaperTrailLumberjack is extremely simple to use. Logging is as simple as calling CocoaLumberjack's various logging statements, once you have a Papertrail logger configured and added to it.

<div class="alert alert-info" role="alert">
  <div class="fa fa-info-circle alert-icon"></div>
  <div class="alert-message">Papertrail does not enable plaintext TCP by default. If TLS is disabled, visit <a href="https://papertrailapp.com/account/destinations">Log Destinations</a>, click <strong>Edit Settings</strong>, and enable plaintext TCP logging.</div>
</div>

### Objective-C

```objc
RMPaperTrailLogger *paperTrailLogger = [RMPaperTrailLogger sharedInstance];
paperTrailLogger.host = @"logsN.papertrailapp.com"; //Your host here
paperTrailLogger.port = XXXXX; //Your port number here    
[DDLog addLogger:paperTrailLogger];
DDLogVerbose(@"Hi papertrailapp.com);
```

By default, logging is via TCP with TLS. To disable TLS, add the following line before adding the logger to DDLog:

```objc
paperTrailLogger.useTLS = NO;
```

To log via UDP instead of TCP, add the following line before adding the logger to DDLog:

```objc
paperTrailLogger.useTcp = NO;
```

If you would like to set either a custom machine name or program name for your log messages, override the following properties:

```objc
paperTrailLogger.machineName = @"CustomMachineName";
paperTrailLogger.programName = @"CustomProgramName";
```

### Swift

```swift
let paperTrailLogger = RMPaperTrailLogger.sharedInstance() as RMPaperTrailLogger!
paperTrailLogger.host = "logsN.papertrailapp.com" //Your host here
paperTrailLogger.port = XXXXX //Your port number here
DDLog.addLogger(paperTrailLogger)
DDLogVerbose("Hi papertrailapp.com")
```

To disable TLS, add the following line before adding the logger to DDLog:

```swift
paperTrailLogger.useTLS = false
```

To log via UDP instead of TCP, add the following line (before adding the logger to DDLog)

```swift
paperTrailLogger.useTcp = false 
```

If you would like to set either a custom machine name or program name for your log messages, override the following properties:

```swift
paperTrailLogger.machineName = "CustomMachineName"
paperTrailLogger.programName = "CustomProgramName" 
```

In both cases, change `XXXXX` and `logsN` to the values shown on [log destinations](https://papertrailapp.com/account/destinations).

<p><a name="method-b-use-cocoaasyncsocket"></a></p>

# CocoaAsyncSocket

Here's an example of how to transmit log data using [CocoaAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket) and its [sendData](https://github.com/robbiehanson/CocoaAsyncSocket/blob/master/GCD/GCDAsyncUdpSocket.h#L479) method:

```objc
GCDAsyncUdpSocket *udpSocket ;
udpSocket = [[GCDAsyncUdpSocket alloc] initWithDelegate:self delegateQueue:dispatch_get_main_queue()];

NSData *data = [
  [NSString stringWithFormat:@"the syslog message will go here"]
  dataUsingEncoding:NSUTF8StringEncoding
];

[udpSocket sendData:data toHost:@"logsN.papertrailapp.com" port:XXXXX withTimeout:-1 tag:1];
```

* Change the `logsN` and `XXXXX` to the values shown on [log destinations](https://papertrailapp.com/account/destinations)
* Generate the actual log message using [stringWithFormat](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/Strings/Articles/FormatStrings.html).

## Log formatting

Papertrail supports both common [syslog formats](/kb/configuration/configuring-remote-syslog-from-embedded-or-proprietary-systems#format). Examples below use the newer RFC 5424 format.

An example syslog string is:

```
<22>1 2014-06-18T09:56:21Z sendername componentname - - - the log message
```

Instead of:

```
[NSString stringWithFormat:@"the syslog message will go here"]
```

do this to get an ISO 8601 timestamp on a device in any locale:

```objc
NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
NSLocale *enUSPOSIXLocale = [NSLocale localeWithLocaleIdentifier:@"en_US_POSIX"];
[dateFormatter setLocale:enUSPOSIXLocale];
[dateFormatter setDateFormat:@"yyyy-MM-dd'T'HH:mm:ssZZZZZ"];
```

and then this to generate the message:

```objc
[NSString stringWithFormat:@"<22>1 %@ some-app some-component - - - the log message",[dateFormatter stringFromDate:[NSDate date]]];
```

Of course, parts of this can be reused across multiple calls or log messages, and variables can be used to provide the app and component names and log message contents.

# Choosing sender name

In addition to the log message, your code provides 2 values to Papertrail: the sender name and the program/component name.

We recommend using one or a small number of distinct values for
the sender name (which is `some-app` in the example above). Most iOS apps
are deployed on tens of thousands of devices, and with that many, the
device isn't really the most meaningful identifier. Having tens of thousands
of unique senders in Papertrail doesn't do anything except clutter the
interface.

Instead, use a sender name which is **not** device- or user-specific.

If you have a user-specific value (such as a user ID, device UUID, or IP
address), use the component/program name for that value.

Here's a sample message which uses the iOS version as the sender name and your
app's own user ID for the component/program name:

```
<22>1 2014-06-18T09:56:21Z iOS-5.1 user-123456789 - - - the log message
```

Or a simpler example which uses the app name as the sender:

```
<22>1 2014-06-18T09:56:21Z my-app user-123456789 - - - the log message
```

The user ID will still be fully searchable in Papertrail, and your internal
staff dashboard can even [link to requests](/kb/how-it-works/linking-to-logs/)
from a given user. In the example above, your dashboard could link to the
query `program:user-123456789` to see logs generated by that user's
device(s).
