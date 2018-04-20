---
layout: default
category: configuration
title: Configuring centralized logging from Kubernetes
sidebar_title: Kubernetes
order: 125
---

# Configuring centralized logging from Kubernetes

To send logs from applications running in a Kubernetes cluster, choose a logging method
based on your setup and deployment preferences. For Google Container Engine, see [GKE logging](/kb/hosting-services/google-cloud-platform#docker-containers-on-google-container-engine).

* [Application-level logging](#application-level-logging)
* [Dedicated logging container](#dedicated-logging-container)
* [System-level logging](#system-level-logging)

# Application-level logging

Use one of Papertrail's suggested configurations for the language or framework (such as [Rails](/kb/configuration/configuring-centralized-logging-from-ruby-on-rails-apps/#method-b-use-the-remote-syslog-logger-gem), [Java](/kb/configuration/java-logback-logging/), [PHP](/kb/configuration/configuring-centralized-logging-from-php-apps/#method-b-send-remote-syslog-from-php-app), [Node](/kb/configuration/configuring-centralized-logging-from-nodejs-apps/#method-b-send-events-from-nodejs), or [.NET](/kb/configuration/nlog-net-logging/)) to log directly from the application. This method doesn't capture logs at the level of the cluster or node, but is adequate for many setups.

# Dedicated logging container

## Sidecar container

A fairly straightforward option when capturing logs for an application container and cluster is to use a logger "sidecar" container within the [Pod](https://kubernetes.io/docs/user-guide/pods/), so that the primary container(s) run the app, and the logging container is a somewhat separated concern. Most commonly, the sidecar connects to a socket or mounted volume on the application containers and runs a logger such as `fluentd`, `logspout`, or `remote_syslog2`. 

To use this method, add a second entry to the `containers` section for the [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/). Since Kubernetes writes the kubelet and container logs (usually Docker logs) to `journald` if available, and otherwise to `/var/log/*.log` (see [Logging and Monitoring Cluster Activity](https://kubernetes.io/docs/concepts/cluster-administration/logging/)), that directory is used as an example throughout.

The approach for a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) (the preferred way to manage Pod state) also uses a sidecar container, but requires adjustments for some loggers to prevent log duplication.

Ensure that the logger sidecar container mounts the Docker container logs directory in the `volumeMounts` section:

```yaml
- name: ...
  image: ...
  volumeMounts:
  - name: dockercontainers
    mountPath: /var/lib/docker/containers
  - name: varlog
    mountPath: /var/log
volumes:
- name: dockercontainers
  hostPath: 
    path: /var/lib/docker/containers
- name: varlog
  hostPath: 
    path: /var/log
```

Fill in the `...` entries with the desired name and relevant [logger](#configuring-the-desired-logger) image. In some setups, the logger container may need to run in privileged mode. If permissions errors occur for access to the Docker container logs, add

```yaml
securityContext:
  privileged: true
```

above `volumeMounts`.

## DaemonSet

Running a [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) is a more complex but potentially more resilient option, and is typically more suitable for Deployments. As with the sidecar container, the DaemonSet may need be run in privileged mode. DaemonSets also require the use of the beta Kubernetes API. This example configuration uses host networking to simplify communication between the DaemonSet and the application container, and mounts a shared Docker socket:

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: ...
spec:
  template:
    metadata:
      labels:
        name: ...
    spec:
      hostPID: true
      hostIPC: true
      hostNetwork: true
      containers:
        - resources:
          securityContext:
            privileged: true
          image: ...
          name: ...
          volumeMounts:
            - name: log
              mountPath: /var/run/docker.sock
      volumes:
        - name: log
          hostPath:
              path: /var/run/docker.sock
```

Fill in the `...` entries with the desired name and relevant [logger](#configuring-the-desired-logger) image.

## Configuring the desired logger

### Logspout 

Logspout is the easiest to set up, but the least flexible: it reads only from the Docker socket. It only requires specifying the image to use and the Papertrail host and port details:

```yaml
env:
  - name: ROUTE_URIS
    value: syslog+tls://logsN.papertrailapp.com:XXXXX
image: gliderlabs/logspout
```

Substitute your own Papertrail [log destination details](https://papertrailapp.com/systems/setup) for `logsN` and `XXXXX` and you're good to go.

The Papertrail host and port details can also be supplied as `args:` rather than through the `env:` variable `ROUTE_URIS`. Note that the container already has a `command:`, so only `args:` should be supplied. Optionally, custom sender and program names can be set with the env variables `SYSLOG_TAG` and `SYSLOG_HOSTNAME`. One example using custom Kubernetes values is:

{% raw %}
```yaml
env:
  - name: SYSLOG_TAG
    value: '{{ index .Container.Config.Labels "io.kubernetes.pod.namespace" }}[{{ index .Container.Config.Labels "io.kubernetes.pod.name" }}]'
  - name: SYSLOG_HOSTNAME
    value: '{{ index .Container.Config.Labels "io.kubernetes.container.name" }}'
```
{% endraw %}

Another option is to set named environment variables and use those, like this:

{% raw %}
```yaml
- name: SYSLOG_TAG
  value: '{{ index .Container.Config.Env "PROGRAM_NAME" }}'
```
{% endraw %}

When using logspout as a sidecar, the Pod or Deployment should not run multiple replicas; instead, use a [DaemonSet](#daemonset) for that configuration.

Credit to [filipegiusti](https://github.com/gliderlabs/logspout/issues/176#issuecomment-220034964) and [mashayev](https://github.com/gliderlabs/logspout/issues/205#issuecomment-231124264) for contributions to these configurations.

### remote_syslog2

To run Papertrail's small [remote_syslog2](https://github.com/papertrail/remote_syslog2) log collection daemon as a sidecar or DaemonSet, use or create a Docker image that contains the most recent `remote_syslog2` [release](https://github.com/papertrail/remote_syslog2/releases) for the container's base OS. To configure logging, map the container and host port needed for the Papertrail destination, and include a mapping for the directory (or directories) where the application's files are found, as well as the Docker logs directory. There are a number of ways to implement the configuration, including copying a `log_files.yml` into the container's `/etc` directory. A more flexible option is a [ConfigMap](https://kubernetes.io/docs/user-guide/configmap/), as shown in this sample configuration:

```yaml
- name: ...
  image: ...
  volumeMounts:
  - name: varlog
    mountPath: /var/log
  - name: dockercontainers
    mountPath: /var/lib/docker/containers
  - name: config-volume
    mountPath: /etc/rs2
volumes:
- name: varlog
  emptyDir: {}
- name: dockercontainers
  hostPath: 
    path: /var/lib/docker/containers
- name: config-volume
  configMap:
    name: rs2-config
```

Fill in the `...` entries with the desired name and relevant `remote_syslog2` image. A bare-bones `log_files.yml` that can be used to create the ConfigMap is:

```yaml
files:
  - /var/log/*.log
  - /var/lib/docker/containers/*/*.log
destination:
  host: logsN.papertrailapp.com
  port: XXXXX
  protocol: tls
```

Replace `logsN` and `XXXXX` with the details from the Papertrail [log destination](https://papertrailapp.com/account/destinations).

Any other log files that the application uses can be added as separate items under `files:`.

When using a ConfigMap, the Docker container `CMD` or `ENTRYPOINT` should point to the mount location for the resulting config file. In the example, the command to run would be:

```shell
$ /usr/local/bin/remote_syslog -D -c /etc/rs2/log_files.yml
```

### Fluentd

[Fluentd](http://www.fluentd.org/) is the most flexible, but also the most complex, logger to set up, and is commonly used in other Kubernetes logging configurations. To use it most effectively with Papertrail, start with a base `fluentd` image, and install the `kubernetes_remote_syslog` plugin:

```shell
$ sudo -u fluent gem install fluent-plugin-kubernetes_remote_syslog
```

This [gem](https://github.com/georgegoh/fluent-plugin-kubernetes_remote_syslog) allows output of logs to a remote syslog destination with some additional setup for Kubernetes.

To configure Fluentd, either create a `fluentd.conf` file in the container, or use a ConfigMap:

```yaml
- name: ...
  image: ...
  imagePullPolicy: IfNotPresent
  securityContext:
    privileged: true
  env:
  - name: FLUENTD_CONF
    value: fluentd.conf
  volumeMounts:
  - name: dockercontainers
    mountPath: /var/lib/docker/containers
  - name: varlog
    mountPath: /var/log
  - name: config-volume
    mountPath: /fluentd/etc
volumes:
- name: dockercontainers
  hostPath: 
    path: /var/lib/docker/containers
- name: varlog
  hostPath: 
    path: /var/log
- name: config-volume
  configMap:
    name: fluentd-config
```

There are two key configurations in the `fluentd.conf` file, one to read the Docker log files and other existing log files:

```xml
<source>
  @type tail 
  path /var/lib/docker/containers/*/*.log
  pos_file /var/log/fluentd-docker.pos
  tag docker.*
  format json
  read_from_head true
</source>
```

and one to output them to Papertrail:

```xml
<match docker.**>
  @type kubernetes_remote_syslog
  host logsN.papertrailapp.com
  port XXXXX
  tag kubernetes-docker
  output_include_tag false
  output_include_time false
</match>
```

In the output section, replace `logsN` and `XXXXX` with the details from the Papertrail [log destination](https://papertrailapp.com/account/destinations).

The full scope of Fluentd [configuration](http://docs.fluentd.org/v0.12/articles/config-file) is beyond the scope of this article, but essentially, this reads in existing log files live, starting from the top using `read_from_head` and tracking position with the `pos_file`). After reading, the output's `docker.*` tag is matched by the `match` directive and output using the `kubernetes_remote_syslog` plugin. That plugin is configured with the Papertrail host and port details, a custom tag, and two output settings that suppress redundant placements of time and tag (program) information. Similar stanzas can be used to read other local files such as `/var/log/*.log`.

This configuration doesn't attempt to augment or format the JSON of the Docker logs, but Fluentd is highly configurable if further processing or formatting is of interest. Check out the potentially useful gem [fluent-plugin-kubernetes_metadata_filter](https://github.com/fabric8io/fluent-plugin-kubernetes_metadata_filter), which allows annotating the containers' Docker logs with useful metadata, or feel free to [contact us](/) to discuss other options.

Fluentd can effectively run as a sidecar even when the Pod or Deployment runs multiple replicas.

# System-level logging

If you have full control over the configuration of the system's logging, configuring a logger at the system level is also an option, although not a suggested Kubernetes configuration because it runs outside the view of the Kubernetes components. This type of setup would most commonly use the [system syslog](/kb/configuration/configuring-remote-syslog-from-unixlinux-and-bsdos-x/) or [remote_syslog2](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix/) on the node. For CoreOS, check the [systemd instructions](/kb/configuration/configuring-centralized-logging-from-systemd/).

Another low-level logging configuration option is to modify the log driver of the Docker daemon:

```shell
$ dockerd --log-driver=syslog --log-opt syslog-address=tcp+tls://logsN.papertrailapp.com:XXXXX
```

As usual, replace `logsN` and `XXXXX` with your log destination details. With these options, any container spun up will use the syslog driver, rather than Docker's default log driver. This has a similar effect to using `--log-opt` at the container level, which is not currently possible in Kubernetes.

To configure system-level logging and keep using `journalctl`, but still get useful host and program labels:

```shell
$ dockerd --log-driver=journald --log-opt labels=SYSLOG_IDENTIFIER
```
  
# Still stuck?

If none of these options seems suitable, [get in touch](/). We love to help.