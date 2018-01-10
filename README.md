# Ghost2logger README

This pack allows StackStorm to receive Syslogs directly and create events from them based on the use of rules.

__WARNING__

Version 1.1.0 is not backwards compatible with any other version. PLEASE ensure you re-generate your configuration. The rules will be fine.

__Usage__

Users send their device and software generated syslogs to a service that is part of this pack.
When rules are created relating to the 'Ghost2logger' syslog pack (this pack!), then those rules are injected into the Ghost2logger syslog service. Through regular expression pattern matching, syslogs that are received are compared against an in memory database. If the transmission host IP address and the regular expression find a match, a trigger is dispatched containing three things:

*	trigger.message
*	trigger.pattern
*	trigger.host

Your actions can then use the trigger fields like any other pack.

Note: StackStorm itself is not acting as the parser for messages. Only messages that have been parsed and matched by the Ghost2logger service are passed in to StackStorm. In essence it's a double match with only 'direct hits' reaching the StackStorm rule engine.

# To Install

```st2 pack install ghost2logger```

Create an ST2 API-Token

```st2 apikey create -k -m '{"used_by": "Ghost2logger"}'```

When the token is generated, make a note of it.

Configure the Ghost2logger pack

```st2 pack config ghost2logger```

You can accept the defaults barring the apikey.

This config file ```ghost2logger.yaml``` which lives in the StackStorm ```/opt/stackstorm/configs``` directory is nearly all defaults. Remember, change the ```st2_api_key``` to the one just generated.

```yaml
---
  username: "admin"
  password: "admin"
  st2_api_key: "ODQzNmU3ZWVhNWNiYmQ1NTllZTNmMDc3NjkzZDE3ZjRiMDNhODQyMDE3YzlmYzA2MjVjNDE0YWU4NGJhNDhmMg"
  syslog_listen_port: "514"
  sensor_listen_ip: "0.0.0.0"
  sensor_listen_port: "12022"
  ghost_ip: "0.0.0.0"
  ghost_port: "12023"
  st2url: "http://127.0.0.1:9101/v1/rules/?limit=10&pack=ghost2logger"
  debugmode = false
  web_hook_auth_header_key: "Authorization"
  web_hook_auth_header_val: "Basic YWRtaW46YWRtaW4="
```

__Breakdown of config items__

* username:                  This is the username to access the API on the Ghost2logger service. It's also the username for the ghost2logger sensor that receives the updates from Ghost2logger. 
* password:                  This is the password to access the API on the Ghost2logger service. It's also the username for the ghost2logger sensor that receives the updates from Ghost2logger.
* st2_api_key:	           This key is so the loopback sensor can access the ST2 API and pull rule information.
* syslog_listen_port:        This is the port that the Syslog listener will open.
* sensor_listen_ip:          This is the IP address of the event sensor. 0.0.0.0 is fine for local installs.
* sensor_listen_port:        This is the port that the event sensor listens on.
* ghost_ip:                  This is the listening IP address of the Ghost2logger service API. This is so the loopback sensor can post rule info.
* ghost_port:                This is the listening port on the IP address above.
* st2url:                    A pre-made URL for the loopback sensor to call to retireve rules.
* debugmode:                 This increases the logging verbosity for the Ghost2logger service.
* web_hook_auth_header_key:  This is the webhook auth header key. Ghost2logger can be used generically without ST2. 
* web_hook_auth_header_val:  The value of the header above. By default, it equates to "admin:admin". This value actually matches the username and password combination in this configuration, as the sensor uses those config fields for authorization against the REST API.

# Ghost2logger Service

As the Ghost2logger component is written in Golang (hence the name - Go ST2 Logger - Ghost looks far cooler), we can run the binary in one of several ways:

*	In the foreground under 'screen'. Start the service running then dettach
*	Run in the foreground with the correct permissions just for the hell of it
*	As a systemd service. Copy the /opt/stackstorm/packs/ghost2logger/bin/ghost2logger.service to /etc/systemd/system
	Check that the service can be read: ```systemctl is-active ghost2logger.service```
	Start the service ```systemctl start ghost2logger.service```
	Check that it's running ```systemctl status ghost2logger.service```

Then you can look at messages coming out of it using: ```journalctl -u ghost2logger.service -f```
Use Ctrl+c to exit without affecting the service.

Ensure that you pass in the configuration with the config switch `-cfg X` when starting Ghost2logger with the systemd backend. Ghost2logger will check the local directory for a configuration and if one is present it will not look anywhere else. Control that behaviour with the `-cfg` switch using a full path to the configuration file.

Check out the service descriptor file in the binary directory of this repository for more info.

# Rules

In order to use Ghost2logger with StackStorm, all you need to do is two things:

*	Point your Syslog senders to the StackStorm instance and port 514
*	Create rules that look like the following:

```yaml
name: rule_1
pack: ghost2logger
ref: ghost2logger.rule_1
criteria:
    trigger.host:
        pattern: 192.168.16.1
        type: eq
    trigger.pattern:
        pattern: thing [0-9]$
        type: eq
enabled: true
tags: []
trigger:
    parameters:
    ref: ghost2logger.pattern_match
    type: ghost2logger.pattern_match
type:
    parameters:
    ref: standard
uid: rule:ghost2logger:rule_1
action:
    parameters:
        channel: '#general'
        message: 'Bot here, I''ve got some news!


            Message: {{trigger.message}}

            Pattern:    {{trigger.pattern}}

            Host:        {{trigger.host}}'
    ref: chatops.post_message
```

Here's some important and interesting stuff to know about rule generation.

1. Since version 1.1.0, you can now use regular expression pattern matching on the `Host` field.
2. `Pattern` field match criteria must always be `eq` as it's always processed as a regular expression in the pipeline. If you do a regular expression match on a regular expression, it gets a little too inception like and just confuses things.
3. You can use either `eq` or `regex` for IP addresses. Both work. Use the first for a direct match, although you can also use `regex`. The heavy processing is done away from StackStorm and thus, any trigger generated will always match the rule critera 1:1. The match criteria for the Ghost2logger service comes from the rule base, so any match is always a direct match and there are no mishits so to speak.

This pack is still work in progress. I'm happy it's at a stable level, but will continue to tweak and add improvements. The 1.1.0 release is a significant step forward to full production grade and please continue to report issues, bugs or feature requests.

Please [look here](http://ipengineer.net/2017/05/stackstorm-ghost2logger-pack/) for a detailed blog post of how to install and configure the pack, including steps of how to restart the StackStorm sensor container, check logs and also how to start and check logs for the Ghost2logger executable. A great place to start for those just starting out with StackStorm. *Note - this blog post as of December 2017 is out of date, please use caution!*

[Email me](mailto:david.gee@ipengineer.net) for more information or if you want to contribute.
You can also check [this link](https://youtu.be/vr_SNLAa0Zw) out which shows the Ghost2logger pack working along with installation.

