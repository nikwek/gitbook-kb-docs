---
layout: default
category: configuration
title: Configuring remote syslog from routers, switches, & network devices
sidebar_title: Routers & network hardware
order: 45
---

# Configuring remote syslog from routers, switches, & network devices

Configure logging on network devices based on Cisco IOS, PIX-OS (ASA), and other network device operating systems.


# Papertrail Setup

Papertrail supports two ways of identifying a device:

* logging to a user-specified syslog port, which is supported by most device operating systems. See [Add Systems](https://papertrailapp.com/systems/setup) to obtain the destination hostname and port. No other changes to Papertrail are required.
* logging to the standard syslog port (514). For this method:

{% include "../\_includes/add_systems_port_514_navigation_steps.md" %}

These two methods cover nearly all network devices. If neither are suitable,
please [let us know](/).

# Device Setup

Don't see your device here? If it can send logs, Papertrail almost certainly can receive them. [Here's how](#other-device).

* [Aruba Mobility Controller](#aruba-networks-mobility-controller)
* [Barracuda Spam Firewall](#barracuda-spam-firewall)
* [Cable/DSL Modems, Wireless Routers](#cabledsl-modems-wireless-routers)
* [Cisco IOS](#cisco-ios)
* [Cisco ASA and PIX](#cisco-asa-and-pix)
* [Cisco CatOS](#cisco-catos)
* [Cisco Meraki](#cisco-meraki)
* [DD-WRT](#dd-wrt)
* [F5 BIG-IP (TMOS)](#f5-big-ip-tmos)
* [Fortigate FortiOS](#fortigate-fortios)
* [Hitachi SAN (HDS VSP)](#hitachi-san-hds-vsp)
* [Juniper Junos](#juniper-junos)
* [Juniper NetScreen](#juniper-netscreen)
* [MikroTik RouterOS](#mikrotik-routeros)
* [OpenWrt](#openwrt)
* [Ruckus ZoneDirector](#ruckus-zonedirector)
* [Ubiquiti EdgeMAX](#ubiquiti-edgemax)
* [Ubiquiti Unifi Controller](#ubiquiti-unifi-controller)
* [Vyatta VyOS](#vyatta-vyos)
* [ZyXEL ZyWALL](#zyxel-zywall)

# Aruba Networks Mobility Controller

{% include "../\_includes/add_systems_port_514_only.md" %}

Papertrail will provide a hostname to use with the Aruba controller's "logging" [command](http://community.arubanetworks.com/t5/Controller-Based-WLANs/How-to-configure-syslog-setting-on-Aruba-Controllers/ta-p/185690). For example:

```
configure terminal
logging logs.papertrailapp.com
exit
write memory
```

If the device doesn't accept a DNS name, replace `logs.papertrailapp.com` with its IP address from `nslookup.`

More: [Aruba log verbosity](http://community.arubanetworks.com/t5/Controller-Based-WLANs/How-to-configure-syslog-setting-on-Aruba-Controllers/ta-p/185690)

# Barracuda Spam Firewall

Barracuda Spam Firewall can send its Mail Syslog (SMTP activity)
and Web Syslog (GUI activity) to Papertrail.

Per [Syslog and the Spam Firewall](https://techlib.barracuda.com/display/bsfv51/syslog+and+the+barracuda+spam+firewall#),
browse to **Advanced**, then **Troubleshooting**. As of this writing,
Barracuda Spam Firewall supports non-default syslog ports but only
supports logging to a destination IP address, not a DNS hostname.
To log to Papertrail, use the settings shown on [Add Systems](https://papertrailapp.com/systems/setup).
{% include "../\_includes/add_systems_ip_address_only.md" %}

Configure each of the 2 message types, like this:

![Barracuda Spam Firewall syslog logging configuration](/assets/images/barracuda_spam_firewall.png)


# Cable/DSL Modems, Wireless Routers

Most home wireless access points and cable/DSL routers can be configured to transmit events.  In the device's Web management interface, set the log or event destination to the hostname and port provided by Papertrail. If the device can only log to
the default syslog port, 514, visit [Add Systems](https://papertrailapp.com/systems/setup) and click
the "[Sender requires port 514](https://papertrailapp.com/systems/new)" link.

# Cisco IOS

To send from Cisco IOS-based devices, connect via SSH or telnet and run `enable` to become administrator.  Enter the following:

```
configure terminal
logging host logsN.papertrailapp.com transport udp port XXXXX
logging facility syslog
logging trap debugging
exit
write memory
```

Replace `logsN` and `XXXXX` with the details provided by Papertrail in [log destinations](https://papertrailapp.com/account/destinations). Most IOS releases after 12.2 support user-supplied ports. The configuration assumes that the router has been told about DNS servers.

For older IOS versions which only support logging to the default port, the configuration could be:

```
logging logs.papertrailapp.com
```

If the device does not have DNS enabled, check the Papertrail account's [log destinations](https://papertrailapp.com/account/destinations) to see which hostname has been assigned, then replace `logsN.papertrailapp.com` with its IP address from `nslookup`.

We recommend the following to make IOS messages interoperate better with the syslog protocol. Disable an extra timestamp and sequence numbers:

```
no service sequence-numbers
no service timestamps debug uptime
no service timestamps log uptime
```

# Cisco ASA and PIX

```
logging enable
logging host outside logsN.papertrailapp.com udp/XXXXX
logging trap informational
logging severity 5
```

`outside` is the name of the Internet-facing interface on the device. Replace `logsN` and `XXXXX` with the details provided by Papertrail in [log destinations](https://papertrailapp.com/account/destinations).

<div class="alert alert-warning" role="alert">
    <div class="fa fa-exclamation-triangle alert-icon"></div>
    <div class="alert-message">Informational and debug log levels can be extremely verbose (often multiple messages per NAT fixup or connection through the device).</div>
</div>

After verifying that logging is functioning, we strongly suggest changing to a less verbose setting like:

```
logging trap notification
```

In devices which support rate-limited logging (such as FWSM), this will rate-limit the log volume to 10 debug-level messages per 30 second interval:

```
logging rate-limit 10 30 level debugging
```

If you explicitly register the device with Papertrail so that it can log to the default syslog port, this will work:

```
logging host outside logs.papertrailapp.com
```

# Cisco CatOS

{% include "../\_includes/add_systems_port_514_only.md" %}

For Cisco Catalyst OS devices, connect via SSH or telnet and run `enable` to become administrator.  Enter the following:

```
set logging server enable
set logging server logs.papertrailapp.com
set logging level all 5
set logging server severity 6
```

## Device doesn't have DNS enabled?

{% include "../\_includes/add_systems_ip_address_only.md" %}

# Cisco Meraki

Cisco Meraki supports logging to syslog. Syslog servers can be defined in the Dashboard from **Network-wide** > **Configure** > **General**.

Click the **Add a syslog server** link to define a new server, using the port details from [Add Systems](https://papertrailapp.com/systems/setup). Instead of configuring a hostname (such as `logsN.papertrailapp.com`), resolve that hostname into IP addresses using `nslookup`. Configure the device to log to one of the IP addresses returned by `nslookup`. Finally, select one or more roles that will send logs to Papertrail.

![](/assets/images/meraki-add-syslog.png)

# DD-WRT

The DD-WRT firmware package provides two different methods for configuring syslog to send log messages to Papertrail: the User Interface and via a startup script on boot.

## The User Interface

{% include "../\_includes/add_systems_port_514_only.md" %}

In the DD-WRT Web interface:

1. Choose the "Services" tab. Enable the "Syslog" service.
2. Enter the hostname provided above, such as `logs.papertrailapp.com`.

### Device requires an IP address, not a hostname?

{% include "../\_includes/add_systems_ip_address_only.md" %}

## Configure Syslog on Boot

To configure syslog to use a port other than 514, create a [startup script](https://www.dd-wrt.com/wiki/index.php/Startup_Scripts) via the router's telnet/SSH connection and enter the following set of commands:

```shell
$ killall syslogd
$ /sbin/syslogd -l <SEVERITY> -L -R <LOG DESTINATION IP ADDRESS>:XXXXX
```

Check the Papertrail account's [log destinations](https://papertrailapp.com/account/destinations) to see which hostname has been assigned, then replace `XXXXX` with the port, and `<LOG DESTINATION IP ADDRESS>` with the hostname's IP address from `nslookup`.

DD-WRT firmware [versions](https://www.dd-wrt.com/wiki/index.php/Index:FAQ#What.27s_the_difference_between_generic.2C_mini.2C_micro_DD-WRT_versions.3F)
other than "micro" can also send security events. To enable security events,
visit the "Security" tab, scroll to "Log Management," and enable desired
options.


# F5 BIG-IP (TMOS)

F5 BIG-IP runs the syslog-ng daemon as its native local log collector. Its syslog-ng can be configured to send to Papertrail. To add Papertrail as the only destination for TMOS logs (using UDP), run:

```
tmsh modify sys syslog remote-servers add {papertrail {host 1.2.3.4 remote-port XXXXX}}
```

Replace `1.2.3.4` with an IP address of the log destination hostname provided by Papertrail. It can be found with `nslookup`. Replace `XXXXX` with the log destination port provided by Papertrail.

More: [syslog in TMOS 9.x/10.x](http://support.f5.com/kb/en-us/solutions/public/5000/500/sol5527.html), [syslog in TMOS 11.x](http://support.f5.com/kb/en-us/solutions/public/13000/000/sol13080.html), [TMOS concepts](http://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/tmos-concepts-11-1-0/tmos_logging.html)


# Fortigate FortiOS

Excerpting from page 29 of [FortiOS Logging & Reporting](http://docs.fortinet.com/fgt/handbook/40mr3/fortigate-loggingreporting-40-mr3.pdf):

To configure FortiOS to log to a syslog server via the management Web interface:

* Go to `Log & Report` > `Log Config` > `Log Setting`
* Select the check box beside `Syslog`
* Select the expand arrow beside the check box to reveal the available options.
* In `Name/IP`, enter the log destination hostname provided by Papertrail
* In `Port`, enter the log destination port provided by Papertrail.
* For `Level`, select a log level the Fortinet unit will log all messages at or above that logging severity level. Popular values are `warning` (4), `error` (3), or `notification` (5).

Alternatively, to configure syslog via the FortiOS command line, run:

```
config log syslogd setting
set status enable
set server logsN.papertrailapp.com
set port XXXXX
end
```

Replace `logsN` and `XXXXX` with the name and port number provided by Papertrail.

More: [FortiOS Logging & Reporting](http://docs.fortinet.com/fgt/handbook/40mr3/fortigate-loggingreporting-40-mr3.pdf), [log message reference](http://docs.fortinet.com/fgt/archives/4.0/techdocs/fortigate-lmr-400.pdf)


# Hitachi SAN (HDS VSP)

{% include "../\_includes/add_systems_ip_address_only.md" %}

## Set syslog server in Storage Navigator

Summarizing [VSP Audit Log User Guide](https://support.hds.com/download/epcra/rd700712.pdf)
section 2-5 ("Transferring audit log files to syslog servers"):

1. Start Storage Navigator and go to Settings > Security > Syslog
2. For "Output to Primary Server," click "Enable"
3. For "Primary Server Setting," type the IP address and port provided by Papertrail
4. For "Location Identification Name," type a name for this array
5. For "Output Detailed Information," click "Enable"
6. Click "Apply"

More: [VSP Audit Log User Guide](https://support.hds.com/download/epcra/rd700712.pdf) (section 2-5 on page 39)


# Juniper Junos

To configure Papertrail in Junos, run:

```
configure
```

to enter configuration mode. Enter these configuration commands, replacing `logsN` and `XXXXX` with the name and port provided by Papertrail:

```
set system syslog host logsN.papertrailapp.com any notice
set system syslog host logsN.papertrailapp.com authorization info
set system syslog host logsN.papertrailapp.com port XXXXX
commit and-quit
```

Confirm the settings with:

```
show system syslog host logsN.papertrailapp.com | display set
```


# Juniper NetScreen

To configure Papertrail in ScreenOS, enter these configuration commands, replacing `logsN` and `XXXXX` with the name and port provided by Papertrail:

```
set syslog config "logsN.papertrailapp.com"
set syslog config "logsN.papertrailapp.com" facilities local7 local7
set syslog config "logsN.papertrailapp.com" port XXXXX
set syslog enable
set syslog backup enable
set log serial-number enable
```


# MikroTik RouterOS

MikroTik RouterOS supports logging to syslog. To configure syslog via the RouterOS command line, run:

```
system logging action add bsd-syslog=yes name=papertrail remote=IP_ADDRESS remote-port=XXXXX target=remote
```

Check the Papertrail account's [log destinations](https://papertrailapp.com/account/destinations) to see
which host has been assigned (it should appear as `logsN.papertrailapp.com`), use nslookup to find its
IP address, then replace `IP_ADDRESS` with that value. Replace `XXXXX` with the port number.

Once that's been configured, send all or nearly all [topics](http://wiki.mikrotik.com/wiki/Manual:System/Log#Topics)
to the newly-created target:

```
system logging add action=papertrail disabled=no prefix="" topics=!async
```

To confirm it, run `/system logging export`. You should see an entry like this

```
/system logging action add bsd-syslog=yes name=papertrail remote=IP_ADDRESS remote-port=XXXXX target=remote
/system logging add action=papertrail topics=!async
```

More: [RouterOS logging actions](http://wiki.mikrotik.com/wiki/Manual:System/Log#Actions), [MikroTik Wiki](http://wiki.mikrotik.com/wiki/Main_Page)


# OpenWrt

To configure OpenWrt to send to Papertrail, connect via SSH and then run the following:

```
uci set system.@system[0].log_ip=IP_ADDRESS
uci set system.@system[0].log_port=XXXXX
uci commit
```

Check the Papertrail account's [log destinations](https://papertrailapp.com/account/destinations) to see which host has been assigned (it should appear as `logsN.papertrailapp.com`), use nslookup to find its IP address, then replace `IP_ADDRESS` with that value.

To confirm the configuration, execute: `uci show system`


# Ruckus ZoneDirector

{% include "../\_includes/add_systems_port_514_only.md" %}

Papertrail will provide a destination hostname for your router to log
to. In the ZoneDirector Web management interface, browse to Configure > System.
Scroll to "Log Settings." Enable "Remote Syslog."
{% include "../\_includes/add_systems_ip_address_only.md" %}

# Ubiquiti EdgeMAX

The EdgeMAX router supports logging to a destination hostname and port. Log in to the router and choose the **System** tab at the bottom of the screen. Look for the _Management Settings_ heading and enter your account's [destination](https://papertrailapp.com/account/destinations) under _System Log_.

![](/assets/images/ubnt-edgemax-system-log.png){: width="443"}

Click **Save** at the bottom and the setting will be applied.

# Ubiquiti UniFi Controller

The UniFi Controller supports logging to a destination hostname and port. Log in to the Controller and choose the **Settings** gear <img src="/assets/images/unifi-controller-settings-gear.4x.png" width="16" height="16" alt="UniFi Controller Settings Gear" style="display:inline; margin-bottom: 0px;" />. Under _Remote Logging_, enter your account's [destination](https://papertrailapp.com/account/destinations) and port.

![](/assets/images/ubnt-unifi-controller-remote-log.png){: width="727"}

Click **Apply Changes** and the destination settings will be pushed to all devices under the UniFi controller's watch including access points, switches, and routers.

# Vyatta VyOS

{% include "../\_includes/add_systems_port_514_only.md" %}

Papertrail will provide a destination hostname for your router to log
to. Provide that hostname to the VyOS router with:

```
set system syslog host <hostname>
```

You may also want to set the log facility and/or level of log messages
which are sent to Papertrail. See
[Brocade Vyatta 5400 manual](http://www.brocade.com/downloads/documents/html_product_manuals/vyatta/vyatta_5400_manual/Basic%20System/wwhelp/wwhimpl/common/html/wwhelp.htm#context=Basic_System&file=Logging.7.20.html) or
[VyOS user guide](http://vyos.net/wiki/User_Guide):


# ZyXEL ZyWALL

To configure ZyWALL to send to Papertrail, connect via SSH or telnet and then
run:

```
enable
configure terminal
logging syslog 1 port XXXXX
logging syslog 1 format cef
logging syslog 1 address logsN.papertrailapp.com
exit
write
exit
```

Replace `XXXXX` and `logsN` with Papertrail-provided values from [log destinations](https://papertrailapp.com/account/destinations).

See [ZyXEL Knowledge Base](http://kb.zyxel.com/KB/searchArticle!gwsViewDetail.action?articleOid=012068&lang=EN).


# Other device

Papertrail supports the industry standard remote syslog protocol,
which is the protocol used by nearly all network devices.

To send logs from a device not shown here, consult the device manual
under "Logging" or "Syslog," or [search Google](https://www.google.com/)
for the device name plus the word "syslog." For example,
`juniper qfx syslog` or `hp procurve syslog`. Most device manufacturers
publish this documentation.

Follow the manufacturer's instructions for remote logging. Use the
Papertrail hostname and port shown on [Add Systems](https://papertrailapp.com/systems/setup).
