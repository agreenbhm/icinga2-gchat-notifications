<!--
  Title: Icinga GChat Notifications
  Description: Icinga 2 notification integration with Google Chat
  Original Authors: nisabek richard.hauswald
  Ported to GChat (from Slack) by: agreenbhm
  -->

# icinga2-slack-notifications (for Google Chat)
Icinga2 notification integration with Google Chat

Original Authors: nisabek richard.hauswald

Ported to GChat (from Slack) by: agreenbhm
  
## Overview

Native, easy to use Icinga2 `NotificationCommand` to send Host and Service notifications to pre-configured Google Chat channel via Webhooks - with only 1 external dependency: `curl`

NOTE: File and variable references to the original plugin (written for Slack) have been left to simplify future modification and/or pull requests.

Also available on [Icinga Exchange](https://exchange.icinga.com/agreenbhm/icinga2-slack-notifications%20%28for%20Google%20Chat%29/releases)

## What will I get?
* Awesome Google Chat notifications:

<p align="center">
  <img src="https://raw.githubusercontent.com/agreenbhm/icinga2-slack-notifications/master/docs/gchat-icinga-notification-example.png" width="600">
</p>

* Notifications inside GChat about your Host and Service state changes
* In case of failure get notified with the nicely-formatted output of the failing check
* Easy integration with Icinga2
* Only native Icinga2 features are used, no bash, perl, etc - keeps your server/virtual machine/docker instances small
* Debian ready-to-use package to reduce maintenance and automated installation effort
* Uses Lambdas!

## Installation 

### Installation using git

1. clone the repository under your Icinga2 `/etc/icinga2/conf.d` directory
 
 `git clone https://github.com/agreenbhm/icinga2-slack-notifications.git /etc/icinga2/conf.d/`

2. Use the `slack-notifications-user-configuration.conf.template` (located in /etc/icinga2/conf.d/icinga2-slack-notifications/src/slack-notifications) file as reference to configure your GChat Webhook URL and Icinga2 Base URL to create your own
 `slack-notifications-user-configuration.conf`
 
 `cp /etc/icinga2/conf.d/icinga2-slack-notifications/src/slack-notifications/slack-notifications-user-configuration.conf.template /etc/icinga2/conf.d/icinga2-slack-notifications/src/slack-notifications/slack-notifications-user-configuration.conf`
 
3. Fix permissions
 
 ```
    chown -R root:nagios /etc/icinga2/conf.d/icinga2-slack-notifications
    chmod 0750 /etc/icinga2/conf.d/icinga2-slack-notifications
    chmod 0640 /etc/icinga2/conf.d/icinga2-slack-notifications/*
 ```

### Configuration 
 
#### Icinga2 features

In order for the slack-notifications to work you need at least the following icinga2 features enabled

`checker command notification`

In order to see the list of currently enabled features execute the following command

`icinga2 feature list`

In order to enable a feature use 

`icinga2 feature enable FEATURE_NAME`

#### Notification configuration

1. Configure Slack Webhook and Icinga2 web URLs in `/etc/icinga2/conf.d/icinga2-slack-notifications/src/slack-notifications/slack-notifications-user-configuration.conf`
```
template Notification "slack-notifications-user-configuration" {
    import "slack-notifications-default-configuration"

    vars.slack_notifications_webhook_url = "<YOUR GCHAT WEBHOOK URL>, e.g. https://chat.googleapis.com/v1/spaces/AAAAAAAA//messages?key=ABCDEFGHIJKLMNOP%3D"
    vars.slack_notifications_icinga2_base_url = "<YOUR ICINGA2 BASE URL>, e.g. http://icinga-web.yourcompany.com/icingaweb2"
}
...
```

2. In order to enable the slack-notifications **for Services** add `vars.slack_notifications = "enabled"` to your Service template, e.g. in `/etc/icinga2/conf.d/templates.conf`

```
 template Service "generic-service" {
   max_check_attempts = 5
   check_interval = 1m
   retry_interval = 30s
 
   vars.slack_notifications = "enabled"
 }
 ```

In order to enable the slack-notifications **for Hosts** add `vars.slack_notifications = "enabled"` to your Host template, e.g. in `/etc/icinga2/conf.d/templates.conf`

```
 template Host "generic-host" {
   max_check_attempts = 5
   check_interval = 1m
   retry_interval = 30s
 
   vars.slack_notifications = "enabled"
 }
 ```

Make sure to restart icinga after the changes

`systemctl restart icinga2`

Note 
> Objects as well as templates themselves can import an arbitrary number of other templates. Attributes inherited from a template can be overridden in the object if necessary.

The `slack-notifications-user-configuration` section applies to both Host and Service, whereas the 
`slack-notifications-user-configuration-hosts` and `slack-notifications-user-configuration-services` sections apply to Host and Service respectively


_Example channel name configuration for Service notifications_

``` 
template Notification "slack-notifications-user-configuration" {
    import "slack-notifications-default-configuration"

    vars.slack_notifications_webhook_url = "<YOUR GCHAT WEBHOOK URL>, e.g. https://chat.googleapis.com/v1/spaces/AAAAAAAA//messages?key=ABCDEFGHIJKLMNOP%3D"
    vars.slack_notifications_icinga2_base_url = "<YOUR ICINGA2 BASE URL>, e.g. http://icinga-web.yourcompany.com/icingaweb2"
}

template Notification "slack-notifications-user-configuration-hosts" {
    import "slack-notifications-default-configuration-hosts"

    interval = 1m
}

template Notification "slack-notifications-user-configuration-services" {
    import "slack-notifications-default-configuration-services"

    interval = 3m
}
```

If you, for some reason, want to disable the slack-notifications from icinga2 change the following parameter inside the 
corresponding Host or Service configuration object/template:

`vars.slack_notifications == "disabled"`

Besides configuring the slack-notifications parameters you can also configure other Icinga2 specific configuration 
parameters of the Host and Service, e.g.:
* types
* user_groups
* interval
* period

## How it works
slack-notifications uses the icinga2 native [NotificationCommand] (https://docs.icinga.com/icinga2/latest/doc/module/icinga2/chapter/object-types#objecttype-notificationcommand) 
to collect the required data and send a message to configured slack channel using `curl`

The implementation can be found in `slack-notifications-command.conf` and it uses Lambdas!

## Testing

To test if notifications are working, go into IcingaWeb2, select a host or service, and choose "Notification" to manually initiate a notification.

## Troubleshooting
The slack-notifications command provides detailed debug logs. In order to see them, make sure the `debuglog` feature of icinga2 is enabled.

`icinga2 feature enable debuglog`

After that you should see the logs in `/var/log/icinga2/debug.log` file. All the slack-notifications specific logs are pre-pended with "debug/slack-notifications"

Use the following grep for troubleshooting: 

`grep "warning/PluginNotificationTask\|slack-notifications" /var/log/icinga2/debug.log`

`tail -f /var/log/icinga2/debug.log | grep "warning/PluginNotificationTask\|slack-notifications"`

## Useful links
- [Setup GChat Webhook](https://developers.google.com/hangouts/chat/how-tos/webhooks#setting_up_an_incoming_webhook)
- [Enable Icinga2 Debug logging](https://docs.icinga.com/icinga2/latest/doc/module/icinga2/chapter/troubleshooting)
- [NotificationCommand of Icinga2](https://docs.icinga.com/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/object-types#objecttype-notificationcommand)
- [Overriding template definitions of Icinga2](https://docs.icinga.com/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/monitoring-basics#object-inheritance-using-templates)
- [Dockerized Icinga2](https://hub.docker.com/r/jordan/icinga2/)
