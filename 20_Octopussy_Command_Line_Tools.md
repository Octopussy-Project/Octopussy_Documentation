# Octopussy Command Line Tools

All the Octopussy command line tools are located in `/usr/sbin/`.

## octo_extractor

```shell
/usr/sbin/octo_extractor [options]
```

octo_extractor extracts desired logs.

You need to specify at least:
  * device (`-ANY-` if you want it all)
  * service (`-ANY-` if you want it all)
  * begin (YYYYMMDDHHMM timestamp)
  * end (YYYYMMDDHHMM timestamp)

Example 1 - Extraction of all logs from the `Octopussy` Service on `octo-devel` Device for May 2014:
```shell
/usr/sbin/octo_extractor --device octo-devel --service Octopussy --begin 201405010000 --end 201405312359
```

Example 2 - Extraction of all logs from the `Sshd` Service on all your Devices for June 2014:
```shell
/usr/sbin/octo_extractor --device -ANY- --service Sshd --begin 201406010000 --end 201406302359
```

Example 3 -  Extraction of all logs from your `octo-devel` Device for July 2014
```shell
/usr/sbin/octo_extractor --device octo-devel --service -ANY- --begin 201407010000 --end 201407312359
```

## octo_replay

```shell
/usr/sbin/octo_replay --device <device> --service <service> --begin YYYYMMDDHHMM --end YYYYMMDDHHMM
````

## octo_reporter

```shell
octo_reporter --report <report> --device <device> --service <service> 
              --loglevel <loglevel> --taxonomy <taxonomy>
              --begin YYYYMMDDHHMM --end YYYYMMDDHHMM [ --pid_param <string> ] --output <output_file>
```

## octo_tool

```shell
/usr/sbin/octo_tool <task> [options]
```

octo_tool is the program to handle Octopussy's tasks.

### octo_tool backup

```shell
/usr/sbin/octo_tool backup <name>
/usr/sbin/octo_tool backup <filename.tgz>
```

`octo_tool backup` command backups all your configuration in a `.tar.gz` file.

That file will contain:
  * Alerts
  * Contacts
  * Devices
  * DeviceGroups
  * Locations
  * Maps
  * Plugins
  * Reports
  * Schedules
  * Search Templates
  * Services
  * ServiceGroups
  * Storages
  * Tables
  * Timeperiods
  * Users
  * and your *system configuration* (Database, LDAP, Nagios NSCA, Proxy, SMTP, XMPP)

If you want a timestamped backup file, use:
```shell
/usr/sbin/octo_tool backup config_saved
```
and it will generate an `config_saved_YYYYMMDDHHMMSS.tgz` backup file.

If you want a strictly named backup file, use:
```shell
/usr/sbin/octo_tool backup octopussy_config.tgz
```
and it will generate an `octopussy_config.tgz` backup file.

### octo_tool cache_clear

```shell
/usr/sbin/octo_tool cache_clear <msgid_stats|taxonomy_stats>
```

### octo_tool message_copy

```shell
/usr/sbin/octo_tool message_copy <msgid_src> <msg_dst>
```

### octo_tool message_move

```shell
/usr/sbin/octo_tool message_move <msgid_src> <msg_dst>
```

### octo_tool service_clone

```shell
/usr/sbin/octo_tool service_clone <servicename> <cloned_servicename>
```

You can easily duplicate an Octopussy Service to make your own and to better fits your need.
If you want to modify the *Linux_Kernel* service to simplify it for example, you can launch that command
```shell
/usr/sbin/octo_tool service_clone Linux_Kernel My_Linux_Kernel
```
and then play with that new service in the Web interface.


### octo_tool table_clone

```shell
/usr/sbin/octo_tool table_clone <tablename> <cloned_tablename>
```

You can easily duplicate an Octopussy Table to make your own and to better fits your need.
If you want to modify the *Mail_Traffic* table to simplify it for example, you can launch that command
```shell
/usr/sbin/octo_tool table_clone Mail_Traffic My_Mail_Traffic
```
and then play with that new table in the Web interface.
