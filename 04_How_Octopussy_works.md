# How Octopussy works

Logs received by your Octopussy server are handled by 3 modules in that order:
  - rsyslog/syslog-ng
  - `octo_dispatcher`
  - `octo_parser`(s)
  
Logs received by your rsyslog/syslog-ng are forwarded to `octo_dispatcher`.

`octo_dispatcher` dispatchs these logs in each device directories like this:

	/var/lib/octopussy/logs/<devicename>/Incoming/<YYYY>/<MM>/msg_<HH>h<MM>_<xx>.log
  
Each `octo_parser`(s) (one `octo_parser` by Device) detects new files in previous directory, 
and if it maches Service(s) pattern(s) defined for this Device, then sort logs from these files in directories like this:
	
	/var/lib/octopussy/logs/<devicename>/<servicename>/<YYYY>/<MM>/msg_<HH>h<MM>.log.gz 

else if it didn't match any Service pattern(s) defined for this Device, it saves logs in these directories:
	
	/var/lib/octopussy/logs/<devicename>/Unknown/<YYYY>/<MM>/msg_<HH>h<MM>.log.gz

**Note:** So, you can not create Service named 'Incoming' or 'Unknown'.
