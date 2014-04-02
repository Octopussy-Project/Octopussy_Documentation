# Configuring Devices to send syslog messages to Octopussy

## Cisco Router/Switch (CatOS)

Connect to your switch, set *enable* mode, and then type 'set logging server enable' and 'set logging server xxx.xxx.xxx.xxx':

    hostname>ena
    Password:
    hostname> (enable) set logging server enable
    hostname> (enable) set logging server xxx.xxx.xxx.xxx

where xxx.xxx.xxx.xxx is the IP Address of your Octopussy Server.

For more information to configure Catalyst Devices, check the [Cisco Documentation](http://www.cisco.com/warp/public/477/RME/rme_syslog.html#topic3).


## Cisco Router/Switch (IOS)

Connect to your switch, set //enable// mode, go to //configure terminal// menu, and then type 'logging on' and 'logging xxx.xxx.xxx.xxx':

    hostname>ena
    Password:
    hostname#conf term
    Enter configuration commands, one per line.  End with CNTL/Z.
    hostname(config)#logging on
    hostname(config)#logging xxx.xxx.xxx.xxx

where xxx.xxx.xxx.xxx is the IP Address of your Octopussy Server.

For more information to configure IOS Devices, check the [Cisco Documentation](http://www.cisco.com/warp/public/477/RME/rme_syslog.html#topic2).


## DenyAll sProxy/rWeb ReverseProxy

Connect to your rWeb/sProxy with ssh and modify *TransferLog*, *ErrorLog* and *SSLLog* lines in httpd.conf for each instance:

    TransferLog "| /usr/bin/logger -t 'sproxy instance_name'"
    ErrorLog    "| /usr/bin/logger -t 'sproxy instance_name'"
    SSLLog      "| /usr/bin/logger -t 'sproxy instance_name'"

*sproxy* will be substituted by *rweb* according to DenyAll system type. (*sproxy* or *rweb*)
Then modify your syslog service to syslog to Octopussy server like any [[install#Linux systems with syslog|Linux System]].


## Fortinet Fortigate Firewall

Connect to your Fortigate WebUI, select the //'Log&Report > Log Config'// menu, then enter the IP Address of your Octopussy Server and the port 514. 


## Juniper DX (RedLine)

Connect to your Juniper DX WebUI, select the //'Admin > Logging'// menu, then enter the IP Address of your Octopussy Server in //Syslog Host 1// or //Syslog Host 2//.



## Linux systems with rsyslog

In your */etc/rsyslog.conf* file, add this line:

    *.*                                                     @@xxx.xxx.xxx.xxx

where xxx.xxx.xxx.xxx is the IP Address of your Octopussy Server.

And then restart the *rsyslog* daemon.

    /etc/init.d/rsyslog restart


## Linux systems with syslog

At the begining of your */etc/syslog.conf* file, add this line:


*.*                                                     @xxx.xxx.xxx.xxx

where xxx.xxx.xxx.xxx is the IP Address of your Octopussy Server.

And then restart the *syslog* daemon.

    /etc/init.d/syslog restart


## Linux systems with syslog-ng

In your */etc/syslog-ng/syslog-ng.conf* file, add your Octopussy server as a new 'destination':

    destination octopussy_server {
        udp( "xxx.xxx.xxx.xxx" port(514) );
    };

where xxx.xxx.xxx.xxx is the IP Address of your Octopussy Server.
Then add this destination to a log directive with the appropriate source:
    log {
        source(all);
        destination(octopussy_server);
    };

For more information to configure syslog-ng, check the *syslog-ng.conf* man page on your system or [here](http://gentoo-wiki.com/MAN_syslog-ng.conf_5).



## NetApp NetCache Proxy

Connect to your NetApp NetCache WebUI, select the //'Setup > System > Logging'// menu, then enter the IP Address of your Octopussy Server.




## PostgreSQL Database

In your *postgresql.conf* file, add **syslog** to your **log_destination** parameter:

    log_destination = 'stderr,syslog'


## Windows Host with Snare Agent

Download and install [Snare Agent for Windows](http://www.intersectalliance.com/projects/SnareWindows/).

By default, Snare use [TAB] as fields delimiter ! :(

You need to change that by editing the *Windows Registry* ! :(

Put ";" in the key *HKEY_LOCAL_MACHINE\SOFTWARE\Intersect Alliance\AuditService\Config\Delimiter*.

Go to the Snare Agent Web Interface, *Network Configuration* menu.
Put your Octopussy Server IP address in the *Destination Snare Server address* field and 514 in the *Destination Port* field.
Then go to the *Apply the Latest Audit Configuration* to apply these changes.

## Xen

Edit your */etc/syslog-ng/syslog-ng.conf* file to get logs from Xen log files and add the prefix "xen: ":

    source s_xen {
        file("/var/log/xen/xend.log" log_prefix("xen: ") follow_freq(15) flags(no-parse));
        file("/var/log/xen/xend-debug.log" log_prefix("xen: ") follow_freq(15) flags(no-parse));
        file("/var/log/xen/domain-builder-ng.log" log_prefix("xen: ") follow_freq(15) flags(no-parse));
        file("/var/log/xen/xen-hotplug.log" log_prefix("xen: ") follow_freq(15) flags(no-parse));
    };

    [...]

    log {
        source(s_xen);
        destination(octopussy_server);
    };
