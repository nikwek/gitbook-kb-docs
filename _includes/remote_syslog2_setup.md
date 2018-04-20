<p><a name="install-remote_syslog"></a></p>

## Install [remote_syslog2](http://github.com/papertrail/remote_syslog2)

Download the current [release](https://github.com/papertrail/remote_syslog2/releases). To extract it and copy the binary into a system path, run:

    $ tar xzf ./remote_syslog*.tar.gz
    $ cd remote_syslog
    $ sudo cp ./remote_syslog /usr/local/bin

RPM and Debian packages are [also available](https://github.com/papertrail/remote_syslog2/releases).

<p><a name="configure"></a></p>

## Configure

Paths to log file(s) can be specified on the command-line, or save [log_files.yml.example](https://github.com/papertrail/remote_syslog2/blob/master/examples/log_files.yml.example) as `/etc/log_files.yml`. Edit it to define:

 * {{ include.path_to_txt }}
 * the destination `host` and `port` provided under [log destinations](https://papertrailapp.com/account/destinations). If no destination port was provided, set `host` to `logs.papertrailapp.com` and remove the `port` config line to use the default port (514).

{% unless include.on_main_remote_syslog2_page == true %}

The [remote_syslog2 README](http://github.com/papertrail/remote_syslog2) has complete documentation and more examples.

{% endunless %}

<p><a name="start"></a></p>

## Start

Start the daemon:

    $ sudo remote_syslog

Logs should appear in Papertrail within a few seconds of being written to the on-disk log file. Problem? See [Troubleshooting](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix/#troubleshooting).

`remote_syslog` requires read permission on the log files it is monitoring.

<p><a name="auto-start"></a></p>

## Auto-start

remote_syslog2 can be automated to start at boot using init scripts ([examples](https://github.com/papertrail/remote_syslog2/blob/master/examples/)) or your preferred daemon invocation method, such as monit or god. See `remote_syslog --help` or the full README on [GitHub](http://github.com/papertrail/remote_syslog2).


{% unless include.on_main_remote_syslog2_page == true %}

## Troubleshooting

See [remote_syslog2 troubleshooting](/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix/#troubleshooting).

{% endunless %}

<hr>
