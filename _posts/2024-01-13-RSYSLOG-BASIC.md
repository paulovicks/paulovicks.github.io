---
title: Rsyslog - Overview
date: 2024-01-13
categories: [Linux]
tags: [logs, logging, syslog, rsyslog] 
---

Linux systems constantly generate logs recording various events, errors, and informational messages. Understanding these logs is crucial for maintaining system health, security, and troubleshooting problems. Here's a basic overview:

Types of Logs:

- System logs: Record general system activity like startup, shutdown, and service status. Typically found in `/var/log/messages`{: .filepath} on older systems or `/var/log/syslog`{: .filepath} on newer ones.
- Application logs: Specific applications create their own logs to track their activities and errors. Locations vary depending on the application.
- Kernel logs: Record kernel-related events like hardware interactions and low-level system activity. Usually found in `/var/log/kern.log`{: .filepath}.
- Audit logs: Capture security-relevant events like user logins, file access, and process executions. Managed by auditd and often stored in `/var/log/audit/audit.log`{: .filepath}.

Rsyslog: The Centralized Logging Daemon

- Rsyslog is a powerful tool for collecting, filtering, and distributing logs from different sources.
- It can read logs from local files, network sockets, and various applications.
- You can configure rules to filter and format logs based on specific criteria.
- Logs can be sent to different destinations like local files, a central log server, or a SIEM (Security Information and Event Management) platform.

Benefits of rsyslog:

- Centralized management: Makes log analysis easier by bringing all logs to one place.
- Filtering and formatting: Focuses on relevant information and standardizes log format.
- Enhanced security: Can centralize and secure sensitive logs like audits.
- Scalability: Handles large volumes of logs generated by complex systems.

Basic rsyslog Usage:

- Configure rules: Edit `/etc/rsyslog.conf`{: .filepath} to define log sources, destinations, and filters.
  - Add custom .conf files in `/etc/rsyslog.d/`{: .filepath}.
- Restart rsyslog: Apply changes by restarting the service.
  - `systemctl restart rsyslog`
- Test: Use a tool like logger to generate a test message to easily search for. (Note: You will need to be at logger version
2.2.21+ to use options -n and -P)
  - `logger -n <server-ip> -P <port> Your-Cool-Test-Message`

- Analyze logs: Use tools like tail or grep to read log files.
  - SSH into your centralized server or SIEM console
  - `tcpdump -nnAs0 -i <interface-name> port <listening-port> | grep Your-Cool-Test-Message`

#### Basic /etc/rsyslog.conf
```text
#### MODULES ####

module(load="imuxsock") # provides support for local system logging (e.g. via logger command)
module(load="imklog")   # provides kernel logging support (previously done by rklogd)

#### GLOBAL DIRECTIVES ####

# Use default timestamp format
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

# Include all config files in /etc/rsyslog.d/
$IncludeConfig /etc/rsyslog.d/*.conf

#### RULES ####

# Log all kernel messages to the console.
# Logging much else clutters up the screen.
kern.*                                                 /dev/console

# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none                /var/log/messages

# The authpriv file has restricted access.
authpriv.*                                              /var/log/secure

# Log all the mail messages in one place.
mail.*                                                  -/var/log/maillog

# Log cron stuff
cron.*                                                  /var/log/cron

# Everybody gets emergency messages
*.emerg                                                 :omusrmsg:*

# Save news errors of level crit and higher in a special file.
uucp,news.crit                                          /var/log/spooler

# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log
```
In the above example you have a few options for defining your centralized server and for now we
will call these "simple" approaches. 

- Option 1: You could define each line individually by changing the directory
path to the IP of your server and port that your server is listening on.

```text
# The authpriv file has restricted access.
authpriv.* /var/log/secure

# The authpriv file has restricted access.
authpriv.* @@<server-ip>:<port>
```

- Option 2: You can add the following line to the bottom of the default config file and use it as a
"catch all".

```text
*.* @@<server-ip>:<port>
```

> When defining your centralized server or SIEM, @ means UDP and @@ means TCP
{: .prompt-info }

Now, best practice is to define separate config files, sometimes called "drop-in files", into the `/etc/rsyslog.d/`{: .filepath}
directory. These days config files can grow very large and using these drop-in files not only makes it easier to read the config,
but it also allows you to organize your logging files.

Some basic examples might be:
```text
# /etc/rsyslog.d/audit.conf
# Send events with a specific key to the server or SIEM
:msg,contains,"key=\"mykey-"  @@<server-ip>

# /etc/rsyslog.d/custom-app.conf
# Send output from a script to specific directory then have rsyslog read that directory

# /etc/rsyslog.d/remote-forwarder.conf
# A middleware collector server that forwards logs from multiple sources to the server or SIEM

```