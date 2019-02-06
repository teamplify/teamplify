# Teamplify runner

[![image](https://travis-ci.org/teamplify/teamplify-runner.svg?branch=master)](https://travis-ci.org/teamplify/teamplify-runner)


# WIP, NOT READY YET

The planned release date of the on-premise version of 
[Teamplify](https://teamplify.com) is February 2019. If you see this message, 
it means that it's not ready yet and that the installer will not work. If 
you're interested in trying Teamplify on-premise, please drop us a line 
at [support@teamplify.com](mailto:support@teamplify.com) and we'll be happy to 
send you an update once it's ready.

-----

  - [About](#about)
  - [System requirements](#system-requirements)
  - [Installation](#installation)
  - [Configuration](#configuration)
  - [Configuration file locations](#configuration-file-locations)
  - [Starting and stopping the service](#starting-and-stopping-the-service)
  - [Upgrades](#upgrades)
  - [Backup and restore](#backup-and-restore)
  - [Maintenance script](#maintenance-script)
  - [Troubleshooting](#troubleshooting)
  - [License](#license)


# About

[Teamplify](https://teamplify.com) is a personal assistant for your development 
team that helps you track work progress and notify your team about situations 
that may require their attention. It is available in two options: 
[in the cloud](https://teamplify.com) or as an on-premise installation. This 
package is the installer and runner for the on-premise version.


# System requirements

Teamplify was designed to run on Linux. It should also work if you wish to 
deploy it on Mac OS X for demonstration purposes. Windows is not supported. 
Before you proceed to the installation, please make sure that your system has 
the following components installed:

  - [Docker version 1.13 and above](https://docs.docker.com/install/);
  - [Python 3.4 and above](https://www.python.org/downloads/).

You can check to see if the required versions are installed with the following 
commands (shown with example output):

``` shell
$ docker -v
Docker version 18.06.1-ce, build e68fc7a215d7133c34aa18e3b72b4a21fd0c6136
$ python3 --version
Python 3.7.2
```

In terms of hardware, we recommend 4GB of RAM, 2 CPU cores and 30 GB of disk 
space (SSD is strongly recommended) as a default server configuration. For most 
small-to-medium organizations (up to a few dozen people), this should be 
enough. Larger workloads, however, may need more resources. The recommended 
strategy is to start with the default server configuration and scale up or 
down depending on the workload.


# Installation

Install the latest version of Teamplify runner with pip:

``` shell
$ pip3 install -U teamplify
```


# Configuration

Teamplify requires a configuration file to run. You can create an initial 
configuration file with the following command:

``` shell
$ teamplify configure
```

This will create a configuration file with default settings in your home
directory: `~/.teamplify.ini`. Now, please use your favorite text editor to 
adjust the contents of this file. You need to specify `product_key` in the 
`[main]` section and `host` and `port` in the `[web]` section. Other parameters 
are optional and can keep their default values.

All configuration options explained:

`[main]`

- `product_key` - the product key of your installation. This is required. 
  Please email us at [support@teamplify.com](mailto:support@teamplify.com) to 
  get the product key;

- `send_crash_reports` - possible values are `yes` and `no`, defaults to `yes`. 
  When set to `yes` the system will automatically send application crash 
  reports to the developer. We recommend keeping this option enabled as it 
  helps us to detect bugs faster and faster ship fixes for them; 

`[web]`

- `host` - domain name on which Teamplify web interface will be running. It 
  must be created in advance and pointing to the server on which you have 
  installed Teamplify;
- `port` - port on which Teamplify web interface will be running;
- `use_ssl` - at the moment, `no` is the only supported value. Built-in SSL 
  support will be available in future versions;

`[db]`

- `host` - defaults to `builtin_db`, which means using the DB instance that is 
  shipped with Teamplify. You can also switch to an external MySQL 5.7 
  compatible database by providing its hostname instead of `builtin_db` and 
  specifying other DB connection parameters below;
- `name` - the database name to use. Must be `teamplify` if `builtin_db` is 
  used;
- `port` - the database port. Must be `3306` for `builtin_db`;
- `user` - DB user. Must be `root` for `builtin_db`;
- `password` - DB password. Must be `teamplify` for `builtin_db`;
- `backup_mount` - a path to a directory on the server which will be mounted 
  into the built-in DB instance container. It is used as a temporary directory 
  in the process of making and restoring backups;

`[email]`

- `address_from` - email address used by Teamplify in the FROM field of its 
  email messages. It could be either a plain email address or an email address 
  with a display name, like this: 
  `Teamplify <teamplify@your-company-domain.com>`;
- `smtp_host` - hostname of an SMTP server used to send emails. Defaults to 
  `builtin_smtp` which means using the SMTP server that is shipped with 
  Teamplify. Built-in SMTP for Teamplify is based on Postfix, and it is 
  production-ready. However, if you plan to use it, we strongly recommend that 
  you add the address of Teamplify's server to the 
  [SPF record](http://www.openspf.org/SPF_Record_Syntax) of the domain used 
  in the `address_from` setting to prevent Teamplify emails from being marked 
  as spam. Or, you can configure Teamplify to use an external SMTP server by 
  providing its hostname instead of `builtin_smtp` and configuring other 
  SMTP connection settings below;
- `smtp_protocol` - SMTP protocol to use. Can be `plain`, `ssl`, or `tls`. 
  Must be `plain` if you use `builtin_smtp`;
- `smtp_port` - SMTP port to use. Must be `25` for `builtin_smtp`;
- `smtp_user` - username for the SMTP server. Must be blank for `builtin_smtp`;
- `smtp_password` - password for the SMTP server. Must be blank for
  `builtin_smtp`;

`[crypto]`

- `signing_key` - the random secret string used by Teamplify for signing 
  cookies and generating CSRF protection tokens. It is automatically generated 
  when you run `teamplify configure`, and typically you don't need to change 
  it unless you think that it may be compromised. In such cases, replace it with 
  another 50-character random string made of Latin characters and numbers 
  (please note that this will force all existing users to log in to the system 
  again).


# Configuration file locations

When you run `teamplify configure`, it will create a configuration file at 
`~/.teamplify.ini`. However, this is not the only possible location. Teamplify 
will search the following locations (listed in the order of their priority, 
from highest to lowest):

1. The location specified in the `--config` parameter in the command line. 
   Example:

   ``` shell
   $ teamplify start --config /path/to/configuration/file
   ```

2. An environment variable named `TEAMPLIFY_CONF`. Example:

   ``` shell
   $ TEAMPLIFY_CONF=/path/to/configuration/file teamplify start
   ```

3. In the home directory of the current user: `~/.teamplify.ini`;
4. At `/etc/teamplify/teamplify.ini`.


# Starting and stopping the service

After you have created the configuration file, start Teamplify with:

``` shell
$ teamplify start
```

During the first run, it may take a while before the application starts since 
it will have to download and configure a bunch of Docker images. Wait for the 
command to complete and open Teamplify in your browser using the `host` and 
the `port` that you provided in the `[web]` section of the configuration. After 
starting the service, it may take a minute or two before it finally comes 
online. If you have problems starting Teamplify, please see the 
[Troubleshooting](#troubleshooting) section below.

If you need to stop Teamplify, run:

``` shell
$ teamplify stop
```

There's also a convenient command to stop the service and start it again. It 
could be useful for applying changes made to the configuration:

``` shell
$ teamplify restart
```


# Upgrades

Teamplify installation consists of the Teamplify runner and the Teamplify 
product itself, which is shipped in the form of Docker images. We follow the 
concept of rolling updates, and so we release updates often (up to a few times 
a week). We recommend that you use the most recent version to keep up with the 
latest features and bugfixes. The upgrade process consists of two steps:

1. Upgrade Teamplify runner:

   ``` shell
   $ pip3 install -U teamplify
   ```

2. Upgrade Teamplify itself:

   ``` shell
   $ teamplify upgrade
   ```

The latter command will automatically detect if a new version has been 
downloaded and will restart the service if necessary. A service restart causes 
a short downtime, so ideally upgrades should be done in periods of low user 
activity. The `upgrade` command restarts the service only when necessary. If no 
upgrade has been downloaded, there will be no restart and therefore no service 
interruption.


# Backup and restore

Teamplify stores your data in MySQL database. As with any other database, it 
might be a good idea to make backups from time to time to ensure that the data 
is not lost in case of a system crash.

To back up the built-in Teamplify database, run:

``` shell
$ teamplify backup [optional-backup-file-or-directory]
```

If launched without parameters, it will make a gzipped backup of the DB and 
store it in the current working directory under a name in the format
`teamplify_<current-date>.sql.gz`, for example, 
`teamplify_2019-01-31_06-58-57.sql.gz`. You have the option of specifying a 
directory or path to the file where you'd like to save your backup.

To restore the built-in Teamplify database from a gzipped backup, run:

``` shell
$ teamplify restore <path-to-a-backup-file>
```

Please note that the commands above will work with the built-in database only. 
If you're running Teamplify with an external database, you'll need to use other 
tools for backups or restores that connect to that database directly.


# Maintenance script

Backing up the data and keeping the software up-to-date are routine operations 
and we recommend automating these processes. Below is a sample script you can 
use to do so.

Create a file named `teamplify-maintenance.sh` with the following contents:

``` shell
#!/usr/bin/env bash

# Backups directory:
BACKUP_LOCATION=/backups/teamplify/

# How many days should we store the backups for:
BACKUP_STORE_DAYS=14

# Back up Teamplify DB and upgrade Teamplify:
mkdir -p $BACKUP_LOCATION && \
    pip3 install -U teamplify && \
    teamplify backup $BACKUP_LOCATION && \
    teamplify upgrade

# If the upgrade was successful, clean up old backups:
if [ $? -eq 0 ]; then
  find $BACKUP_LOCATION -type f -mmin +$((60 * 24 * $BACKUP_STORE_DAYS)) \
      -name 'teamplify_*.sql.gz' -execdir rm -- '{}' \;
fi


# The final step is optional but recommended. Add your code so that would 
# sync contents of $BACKUP_LOCATION to a physically remote location.
#
#   ... add your backups sync code below:
```

In the code above, please adjust the path for BACKUP\_LOCATION and the value 
for BACKUP\_STORE\_DAYS as necessary. At the end of the script, you can add 
your code that would sync your backups to a remote location. This is optional, 
but it's a highly recommended precaution that would help you to recover your 
backup in the case of a disaster. For example, you can use 
[aws s3 sync](https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html) to
upload the backups to AWS S3.

When the maintenance script is ready, make it executable with 
`chmod +x teamplify-maintenance.sh` and set it as a cron job to run daily. Open
the crontab schedule:

``` shell
$ crontab -e
```

Append the following entry (remember to replace the path to the script):

``` shell
0 3 * * * /path/to/the/script/teamplify-maintenance.sh
```

In the example above, it is scheduled to run daily at 3 AM. See 
[cron syntax](https://en.wikipedia.org/wiki/Cron) for a detailed explanation.
When ready, save and close the file.


# Troubleshooting

So what could possibly go wrong?

### Teamplify doesn't start

Please check the following:

- The service won't start if the configuration file is missing or contains 
  errors. In such cases, the `teamplify start` command will report a problem. 
  Please inspect its output;
- There could be a problem with the domain name configuration. If the 
  `teamplify start` command has completed successfully, you should see 
  Teamplify's interface in the browser when you open an address specified in 
  `host` and `port` parameters in the `[web]` section of the 
  [Configuration](#configuration). If that doesn't happen, i.e. if browser says 
  that it can't find the server or the server is not responding, then most 
  likely this is a problem with either the domain name or firewall 
  configuration. Please make sure that the domain exists and points to 
  Teamplify server, and that the port is open in the firewall;
- If you see the "Teamplify is starting" message, you should give it a minute 
  or two to come online. If nothing happens after a few minutes, there could be 
  a problem during application start. Application logs may contain additional 
  information:

  ``` shell
  $ docker logs teamplify_app
  ```

  Please let us know about the problem and attach the output from the command 
  above. You can either 
  [open an issue on Github](https://github.com/teamplify/teamplify-runner/issues), 
  or contact us at [support@teamplify.com](mailto:support@teamplify.com), or 
  use the live chat on [teamplify.com](https://teamplify.com).

### Email delivery issues

Emails can go to spam or sometimes aren't delivered at all. If you're
running a demo version of Teamplify on your desktop at home, this is
very likely to happen, since IPs of home internet providers have a large
chance of being blacklisted in spam databases. We recommend that you
check the following:

- If you're going to use the built-in SMTP server, consider running Teamplify 
  on a server hosted in a data center or at your office, but not at home. Next, 
  please make sure that you've added the IP of Teamplify server to the 
  [SPF record](http://www.openspf.org/SPF_Record_Syntax) of the domain used
  in `address_from` setting in the configuration file;
- Some email providers, for example, Google Mail, explicitly reject emails 
  sent from blacklisted IPs. It might be helpful to examine SMTP server 
  logs to see if that's what's happening:

  ``` shell
  $ docker logs teamplify_smtp
  ```

- Alternatively, if you have another SMTP server that is already configured 
  and can reliably send emails, you can configure Teamplify to use this server 
  instead of the built-in SMTP. See `[email]` section in 
  [Configuration](#configuration) for details;

### Other

For any issue with Teamplify, we recommend that you try to [upgrade](#upgrades) 
first. We release updates frequently. It's quite possible that the problem that 
you encountered is already addressed in a newer version.

If the suggested solutions above don't work, please don't hesitate to 
[open an issue on Github](https://github.com/teamplify/teamplify-runner/issues) 
or contact us at [support@teamplify.com](mailto:support@teamplify.com). You can
also use the live chat on [teamplify.com](https://teamplify.com). We're ready 
to help!


# License

Teamplify runner is available under the MIT license. Please note that the MIT 
license applies to Teamplify runner only, but not to the main Teamplify product. 
Some Docker images downloaded by Teamplify runner will contain a proprietary 
code that is not open source and is distributed under its own 
[terms and conditions](https://teamplify.com/terms/).
