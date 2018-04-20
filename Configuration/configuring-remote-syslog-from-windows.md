---
layout: default
category: configuration
title: Configuring remote syslog from Windows
sidebar_title: Windows
order: 15
---

# Configuring remote syslog from Windows

To send log files and event logs from all Windows variants, we recommend [nxlog](http://nxlog.org/products/nxlog-community-edition). In case nxlog will not run on your machine, [Eventlog-to-Syslog](#eventlog-to-syslog) can be installed on the machine.

# Nxlog

## Installation

1. Download the latest version using the link at the top of [the releases table](http://nxlog.org/products/nxlog-community-edition/download).
2. Double click the downloaded MSI.
3. Follow the on-screen prompts.

## Basic Configuration

1. Open `C:\Program Files (x86)\nxlog\conf\nxlog.conf`, or on 32 bit platforms, `C:\Program Files\nxlog\conf\nxlog.conf`.
2. Replace the contents with [this template](https://gist.github.com/leonsodhi/6688501#file-nxlog_config_template-conf).
3. Replace `<host>.papertrailapp.com` and `YOUR_PORT` with the details shown under [log destinations](https://papertrailapp.com/account/destinations).
4. (Optional) modify `File 'C:\\path\\to\\*.log'` to send the contents of a local log file. For multiple log files in different directories, add more `<Input watchfileN>` blocks and include them in Route 2 near the bottom of the [example config](https://gist.github.com/leonsodhi/6688501#file-nxlog_config_template-conf). The commented out `<Input watchfile2>` block illustrates this process.
5. Restart the nxlog service.

## Encrypted Logging using TCP+TLS (optional)

1. Download [https://papertrailapp.com/tools/papertrail-bundle.pem](https://papertrailapp.com/tools/papertrail-bundle.pem) to the `cert` directory under your nxlog installation location.
2. Add `define CERTDIR %ROOT%\cert` to the top of the file, near the other define statements.
3. Replace the syslogout block with:
   ```xml
   <Output syslogout>
     Module om_ssl
     Host logsN.papertrailapp.com
     Port XXXXX
     CAFile %CERTDIR%/papertrail-bundle.pem
     AllowUntrusted FALSE
   </Output>
   ```
4. Replace `logsN.papertrailapp.com` and `XXXXX` with the details shown under [log destinations](https://papertrailapp.com/account/destinations).
5. Restart the nxlog service.


# Eventlog-to-Syslog

In case nxlog will not run on your machine, Eventlog-to-Syslog can be installed and
configured using the instructions below.

## Download

Download `evtsys-64bit.zip` or `evtsys32bit.zip` from [Google Code](http://code.google.com/p/eventlog-to-syslog/downloads/list).
As of this writing, the current version is 4.5.1.

Download the regular build, not the `Large Packet` build. As noted [here](http://code.google.com/p/eventlog-to-syslog/source/detail?r=50),
the `Large Packet` build changes the maximum packet size from 1500 bytes to 4096 bytes.
The largest packet ([MTU](http://en.wikipedia.org/wiki/Maximum_transmission_unit)) on
the Internet is 1500 bytes, so the regular build is required.

## Install

Extract the .zip file. Copy the 2 extracted files to `C:\Windows\System32` (or your system's equivalent directory).

## Run

1. Start a DOS Prompt as a local administrator: **Start** > right-click on DOS Prompt > **Run as Administrator**.
2. Navigate to `C:\Windows\System32`.
3. Run `evtsys.exe` to install the service, providing the destination host and port from Papertrail's [Add Systems](https://papertrailapp.com/systems/setup) page. For example:

```
> evtsys.exe -i -h logsN.papertrailapp.com -p XXXXX
```

Change the `logsN` and `XXXXX` arguments to match your [Papertrail log destination](https://papertrailapp.com/account/destinations).

This will start the eventlog to syslog relay. Subsequent Windows events should appear in Papertrail within 5 seconds.

Here are the [full arguments](http://code.google.com/p/eventlog-to-syslog/source/browse/trunk/4.0/main.c#144)
and the [readme](http://code.google.com/p/eventlog-to-syslog/downloads/list). 

## Manage

To uninstall the service, run with `-u`, like:

```shell
> evtsys.exe -u -h logsN.papertrailapp.com -p XXXXX
```

Change the `logsN` and `XXXXX` arguments to match your [Papertrail log destination](https://papertrailapp.com/account/destinations).

In addition to the Services control panel, the service can be controlled with:

```shell
> net start evtsys
> net stop evtsys
```