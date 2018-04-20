---
layout: default
category: configuration
title: Configuring centralized logging from ESXi
sidebar_title: VMware ESXi
order: 170
toc: false
---

# Configuring centralized logging from ESXi

VMware best practices dictate that ESXi virtualization hosts should have their logs stored remotely. ESXi supports sending log data to a remote log collector via the syslog protocol, which allows Papertrail to ingest it.

To do so requires opening a new outgoing firewall rule, which can only be performed by SSHing to the ESXi host itself. Once the SSH service has been started via the "Security Policy" section of the VMware client, log in and run the following command to generate a new firewall rule:

```shell
$ cat <<EOF > /etc/vmware/firewall/papertrail.xml
<ConfigRoot>
  <service id='1000'>
    <id>Papertrail</id>
    <rule>
      <direction>outbound</direction>
      <protocol>tcp</protocol>
      <porttype>dst</porttype>
      <port>XXXXX</port>
    </rule>
    <enabled>true</enabled>
    <required>false</required>
  </service>
</ConfigRoot>
EOF
```

where `XXXXX` is the port number shown under [log destinations](https://papertrailapp.com/account/destinations).

Once that's done, refresh the firewall configuration using:

```shell
$ esxcli network firewall refresh
```

then configure remote syslog using:

```shell
$ esxcli system syslog config set --loghost='ssl://logsN.papertrailapp.com:XXXXX'
$ esxcli system syslog reload
```

where `logsN` and `XXXXX` are the name and port number shown under [log destinations](https://papertrailapp.com/account/destinations).

Note that these new rules will not persist across a reboot unless they are applied via a vSphere Installation Bundle (VIB). A custom VIB can be created by following the instructions in [this](http://kb.vmware.com/kb/2007381) VMware Knowledge Base article.
