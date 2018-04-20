---
layout: default
category: configuration
title: Configuring centralized logging from Docker
sidebar_title: Docker
order: 80
---

# Configuring centralized logging from Docker

To send logs from applications running in a Docker container, choose
based on your Docker version and deployment preferences.

Here's a [screenshot](/assets/images/docker-logs.png) of Docker logs
in Papertrail's event viewer.

# Log aggregation methods

Not sure? Use a **[logspout container](#logspout)**.

| I'm...                                           | Use..                                | Notes |
| ------------------------------------------------ | ------------------------------------ | ----- |
| **a typical Docker user**                        | [logspout container](#logspout)      | **Most popular** |
| brand new to Docker                              | [remote_syslog2 & rsyslog](#generic) | Same as non-Docker setup |

If neither of the methods above fit your environment, these are also viable:

* [Docker syslog driver](#log-driver-syslog) (`--log-driver=syslog`; 1.7+ only)
* [Mounted log volume](#mount) (`-v`)

# Logspout

To start a `logspout` container, run:

```shell
$ docker run --restart=always -d \
    -v=/var/run/docker.sock:/var/run/docker.sock gliderlabs/logspout  \
    syslog://logsN.papertrailapp.com:XXXXX
```

Replace `logsN` and `XXXXX` with details from one of your
Papertrail [log destinations](https://papertrailapp.com/account/destinations).

The `--restart` ensures that it will be re-run automatically at host
boot. Docker run behavior is sensitive to the ordering of some arguments,
so be aware of ordering if altering the suggested command.

## Example: logspout with Docker Swarm

This example invocation starts logspout as part of a
[Docker Swarm](https://docs.docker.com/swarm/) cluster. Docker will start one
logspout service per node
"[global](https://docs.docker.com/engine/swarm/services/#/control-service-scale-and-placement)"
mode) and expose `/var/run/docker.sock` from each node:

```shell
$ docker service create --name log \
    --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock \
    --mode global gliderlabs/logspout syslog+tls://logsN.papertrailapp.com:XXXXX
```

<a name="generic"></a>

# remote_syslog2 & rsyslog

Keep it simple. Use Papertrail's non-Docker-specific methods for
sending system and app logs. For example, send app log files with remote_syslog2
and system logs with rsyslog. These run in container(s) and host(s).

This is the same method used by any
other Linux logging configuration, such as for a physical system or Xen VM.

For app logs, follow the steps on
[text log files](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix/).

For system logs, follow the steps on
[systemd](/kb/configuration/configuring-centralized-logging-from-systemd/) or standard [syslog](/kb/configuration/configuring-remote-syslog-from-unixlinux-and-bsdos-x/)
setup.

This works particularly well when your containers are more like traditional
systems, such as when they were migrated directly from dedicated systems.

# Alternatives

<a name="log-driver-syslog"></a>

## syslog driver: `--log-driver=syslog`

Docker can log directly to a syslog destination using:

```shell
$ docker run --log-driver=syslog
    --log-opt syslog-address=[udp|tcp+tls]://host:port
    --log-opt tag=optional-container-name
    image-name
```

See [--log-driver](https://docs.docker.com/engine/admin/logging/overview/)
and [syslog options](https://docs.docker.com/engine/reference/run/#logging-drivers---log-driver).

The destination can be a syslog daemon running on the host, a syslog daemon
running on another container, or Papertrail itself (see [Add Systems](https://papertrailapp.com/systems/setup)
for host and port). For example:

```shell
$ docker run --log-driver=syslog
    --log-opt syslog-address=udp://logsN.papertrailapp.com:XXXXX 
    image-name
```

replacing `logsN` and `XXXXX` with details from the Papertrail [log destination](https://papertrailapp.com/account/destinations) (and `image-name` with the name of the desired image to run).

<div class="alert alert-info" role="alert">
  <div class="fa fa-info-circle alert-icon"></div>
  <div class="alert-message">Using an alternative log driver means that <code>docker logs</code> will no longer display logs. To access logs, go to the syslog daemon or Papertrail.</div>
</div>

### Important: TCP connection failure on startup means container failure

If using TCP/TLS, failure to connect to the syslog endpoint will result in the container not starting. Docker currently prioritizes not losing logs over starting a container without log delivery. See [#21966: If a remote TCP syslog server is down, docker does not start](https://github.com/moby/moby/issues/21966). There are two ways to eliminate or mitigate this issue:

* To ensure that a container is never affected by log delivery issues, use UDP instead of TCP. UDP does not guarantee delivery, but will also never block the container from running.
* To use TCP but reduce the likelihood of log delivery issues affecting the container, use the host-local syslog or a local container, and forward the logs to Papertrail from there.

### Configuring syslog tags

By default, Docker uses the first 12 characters of the container ID to tag log
messages. To choose a different tag or include other attributes in the tag, see
[log tags](https://docs.docker.com/engine/admin/logging/log_tags/).

### Docker Compose

If using Docker Compose, add something like the following as an entry:

{% raw %}
```yaml
logging:
  driver: syslog
  options:
    syslog-address: "udp://logsN.papertrailapp.com:XXXXX"
    tag: "{{.Name}}/{{.ID}}"
```
{% endraw %}

This also shows an example of a custom tag.

<a name="mount"></a>

## Mounted log volume: docker -v

Have the containers log to files in a directory/volume that's shared with
(mounted from) the host. See [Mount a host directory as a data volume](https://docs.docker.com/engine/userguide/dockervolumes/#mount-a-host-directory-as-a-data-volume)
and `docker -v`.

This can be one directory per container or a shared directory for all containers.

Then on the host, use Papertrail's typical techniques for
[sending logs from files](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix).

### Dedicated log collector containers

If you need to keep the host's workload to a minimum or simply
prefer to completely separate unrelated concerns, use
[log collector containers](https://github.com/octohost/remote_syslog) to send
logs (instead of a daemon on the host). This container image does nothing except
watch a log file (exposed as a `docker -v` mount) and send the contents to Papertrail.
