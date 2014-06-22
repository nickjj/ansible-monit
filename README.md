## What is ansible-monit? [![Build Status](https://secure.travis-ci.org/nickjj/ansible-monit.png)](http://travis-ci.org/nickjj/ansible-monit)

It is an [ansible](http://www.ansible.com/home) role to install monit and allow you to easily manage as many processes as you want.

### What problem does it solve and why is it useful?

I wasn't happy with any of the monit roles I came across. They were either missing features that I wanted or didn't work at all.

Here is a feature list of this role:

- Adjust the poll interval
- Optionally set a delay to when monit starts
- Optionally disable alerting by default
- Fully configure the mail details so you can get e-mail alerts
- Fully configure the http interface including optional ssl support
- Load arbitrary configs from the standard /etc/monit/conf.d path
- Create arbitrary monit tasks with the least amount of pain

## Role variables

Below is a list of default values along with a description of what they do.

```
---
# How often in seconds should monit check the process?
monit_interval: 60

# How many seconds delay should monit wait before monitoring on the first time?
monit_start_delay: 30 # 0 to disable

# Where should monit log to?
monit_logfile: syslog facility log_daemon

# Where should events be written in case the mail server is down?
monit_event_path: /var/lib/monit/events

# Should e-mail alerts be sent?
monit_alerts_on: true

# Skip certain types of events for the e-mail alerts.
monit_alert_skip_when: "action, instance, pid, ppid"

# The mail host name.
monit_mail_host: smtp.gmail.com

# The mail port.
monit_mail_port: 587

# The mail username.
monit_mail_username: you

# The mail password.
monit_mail_password: securepassword

# Which mail encryption should be used?
monit_mail_encryption: TLSV1 # SSLAUTO, SSLV2, SSLV3, TLSV1, TLSV11, TLSV12

# Who should receive the alerts?
monit_mail_alert_to: me@mydomain.com

# Who should the sender of the alert e-mail be?
monit_mail_default_from: monit@mydomain.com

# What should the subject be?
monit_mail_default_subject: "[Monit] $SERVICE $EVENT"

# What should the message say?
monit_mail_default_message: "Monit $ACTION $SERVICE at $DATE on $HOST: $DESCRIPTION."

# The host for the web interface.
monit_http_host: localhost

# Which port should it run on?
monit_http_port: 2812

# Which address(es) can access it?
monit_http_allow: ["localhost"]

# Should ssl be used?
monit_http_ssl: false

# If ssl is enabled, what local pem file should it copy?
monit_http_local_pemfile_path: ~/dev/testproject/secrets/monit.pem

# Defaults to nothing but you can add as many monit tasks as you want.
# Don't forget the | to enable text blocks, feel free to use template tags too.
monit_process_list: |
#  check process foo with pidfile /full/path/to/foo.pid
#    start program = "/etc/init.d/foo start"
#    stop program = "/etc/init.d/foo stop"

#  check process bar with pidfile /full/path/to/bar.pid
#    start program = "/etc/init.d/bar start" with timeout 60 seconds
#    stop program = "/etc/init.d/bar stop"
#    if cpu > 80% for 5 cycles then restart
#    if 3 restarts within 5 cycles then timeout

# The amount in seconds to cache apt-update.
apt_cache_valid_time: 86400
```

#### 2 ways of adding monit checks

You can add the checks by overriding `monit_process_list` and supplying a text block as shown above or you can write out configs to `/etc/monit/conf.d` in your own ansible tasks.

## Example playbook without ssl

For the sake of this example let's assume you have a group called **app** and you have a typical `site.yml` file.

To use this role edit your `site.yml` file to look something like this:

```
---
- name: ensure app servers are configured
- hosts: app

  roles:
    - { role: nickjj.monit, tags: monit }
```

Let's say you want to edit a few defaults, you can do this by opening or creating `group_vars/app.yml` which is located relative to your `inventory` directory and then making it look something like this:

```
---
monit_interval: 120

# This would require you to enter myusername/mypassword in a browser while also
# being on localhost.
monit_http_allow: ["localhost", "myusername:mypassword"]
```

## Example playbook with ssl

First things first, make sure you read the section above because setting up the ssl version of this role is the same as the non-ssl version except it requires a few more defaults to be changed.

#### Do you need a pem file?

If you want your monit page to be accessible over ssl then you must generate a pem file.

You must run the following 2 commands:

`$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout monit.pem -out monit.pem`  
`$ openssl gendh 512 >> monit.pem`

Keep note of the path of where you created this file.

#### You are ready to change a few defaults

Overwrite at least the following default value(s) in `group_vars/all.yml`:

```
monit_http_ssl: true

monit_http_local_pemfile_path: PUT_YOUR_PEM_FILE_PATH_HERE
```

## Installation

`$ ansible-galaxy install nickjj.monit`

## Requirements

Tested on ubuntu 12.04 LTS but it should work on other versions that are similar.

## Troubleshooting common errors

#### The monit daemon is not running
In order to run it you must specify an interval for which monit to run at. You also need the web interface to be configured correctly.

#### I can't access the web page with curl with ssl enabled
You need to tell curl to run in insecure mode, `curl https://localhost:2812 --insecure`.

## Ansible galaxy

You can find it on the official [ansible galaxy](https://galaxy.ansible.com/list#/roles/949) if you want to rate it.

## License

MIT