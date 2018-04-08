# Octopussy Installation

**NOTE: This documentation covers Octopussy 1.0.16 and greater. For older versions take a look at [this one](https://github.com/Octopussy-Project/Octopussy_Documentation/tree/1.0.14).**

Please make sure that SELinux, AppArmor, or other security software like these are well configured in order to work with Octopussy and its software (Apache, MySQL, RSyslog, ...)

(cf. Issues [#213](https://github.com/sebthebert/Octopussy/issues/213#issuecomment-28749963), [#305](https://github.com/sebthebert/Octopussy/issues/305#issuecomment-28750893), [#307](https://github.com/sebthebert/Octopussy/issues/307#issuecomment-28750930))

## Installation with Ansible

A lot of work have been done during [Hacktoberfest](https://hacktoberfest.digitalocean.com) 2017 to provide an [Octopussy Ansible Role](https://github.com/Octopussy-Project/ansible-role-octopussy) which covers CentOS, Debian & Ubuntu installation.

I build a complete platform with Ansible & Docker to test Octopussy installation on:
  * CentOS 6 & 7
  * Debian 8 & 9
  * Ubuntu 14.04 & 16.04


## Debian/Ubuntu Installation
*(tested on Debian 8 & 9  and Ubuntu 14.04 & 16.04)*

Get the latest octopussy debian package [here](http://sourceforge.net/project/showfiles.php?group_id=154314|here).


Update your system with 'apt-get update':

```shell
apt-get update
```

Then install Octopussy:

```shell
dpkg -i octopussy_<version>_all.deb
apt-get -f install
```

Enable syslog reception from other hosts in `/etc/rsyslog.conf`:

```
$ModLoad imuxsock # provides support for local system logging
$ModLoad imklog   # provides kernel logging support (previously done by rklogd)
#$ModLoad immark  # provides --MARK-- message capability

# provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514

# provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514
```

and then restart Rsyslog

```shell
/etc/init.d/rsyslog restart
```

Generate self-signed Certificate for Octopussy Web Server:

```shell
openssl genrsa > /etc/octopussy/server.key
openssl req -new -x509 -nodes -sha1 -days 365 -key /etc/octopussy/server.key > /etc/octopussy/server.crt
```
and then restart Octopussy webserver

```shell
/etc/init.d/octopussy web-stop
/etc/init.d/octopussy web-start
```

## CentOS Installation
*(tested on CentOS 6.9 & 7.4)*

### Software Installation

Install software requirements:

```shell
yum install -y epel-release expat-devel gcc make htmldoc httpd \
  mod_perl mod_ssl mysql mysql-devel mysql-server \
  nscd openssl-devel patch perl perl-devel perl-CPAN \
  psmisc rrdtool rsyslog sudo
```

Install Perl modules requirements:

```shell
cpan Apache::ASP App::Info App::Info::HTTPD Authen::SASL Cache::Cache \
  Crypt::PasswdMD5 Data::GUID Date::Manip \
  Time::Hires Test2::Require::Module \
  DateTime::Format::Strptime DBD::mysql DBI \
  Email::MIME Email::Sender Getopt::Long IO::Socket::SSL \
  JSON Linux::Inotify2 List::MoreUtils \
  Locale::Maketext::Lexicon Locale::Maketext::Simple \
  LWP Net::FTP Net::LDAP Net::SCP Net::SSLeay Net::Telnet Net::XMPP \
  Pod::Find Pod::Usage Proc::PID::File Proc::ProcessTable Readonly \
  Regexp::Assemble Sys::CPU Term::ProgressBar Time::Piece \
  Unix::Syslog URI version XML::Simple
```

Get the latest octopussy source package [here](http://sourceforge.net/project/showfiles.php?group_id=154314).
Launch installation script:

```
tar xvfz octopussy-<version>.tar.gz
cd octopussy
bash LINUX/INSTALL.sh
```

### CentOS Configuration

Change iptables configuration (in /etc/sysconfig/iptables) and add this:

```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8888 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 514 -j ACCEPT
-A INPUT -m udp -p udp --dport 514 -j ACCEPT
```

Restart iptables with the new configuration:

```shell
/etc/init.d/iptables restart
```

Disable syslog and enable rsyslog and octopussy at boot time:

```shell
chkconfig --del syslog
chkconfig --add octopussy
chkconfig --add rsyslog
chkconfig --level 2345 octopussy on
chkconfig --level 2345 rsyslog on
```

Disable RequireTTY in sudoers (`/etc/sudoers`):

```
#Defaults    requiretty
```

Modify Rsyslog default configuration (in `/etc/sysconfig/rsyslog`):

```
SYSLOGD_OPTIONS="-c 3"
```

Modify Rsyslog configuration (in `/etc/rsyslog.conf`):

```
	#################
	#### MODULES ####
	#################

	$ModLoad imuxsock # provides support for local system logging
	$ModLoad imklog   # provides kernel logging support (previously done by rklogd)
	#$ModLoad immark  # provides --MARK-- message capability

	# provides UDP syslog reception
	$ModLoad imudp
	$UDPServerRun 514

	# provides TCP syslog reception
	$ModLoad imtcp
	$InputTCPServerRun 514


	###########################
	#### GLOBAL DIRECTIVES ####
	###########################

	#
	# Use traditional timestamp format.
	# To enable high precision timestamps, comment out the following line.
	#
	#$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

	#
	# Set the default permissions for all log files.
	#
	$FileOwner root
	$FileGroup adm
	$FileCreateMode 0640
	$DirCreateMode 0755

	#
	# Include all config files in /etc/rsyslog.d/
	#
	$IncludeConfig /etc/rsyslog.d/*.conf
```

And put your server name in `/etc/rsyslog.d/octopussy.conf`:

```
:hostname, !isequal, "your_server_name" ~
```

Restart rsyslog:

```shell
/etc/init.d/rsyslog restart
```

Generate self-signed Certificate for Octopussy Web Server:

```shell
openssl genrsa > /etc/octopussy/server.key
openssl req -new -x509 -nodes -sha1 -days 365 -key /etc/octopussy/server.key > /etc/octopussy/server.crt
```

and then restart Octopussy webserver

```shell
/etc/init.d/octopussy web-stop
/etc/init.d/octopussy web-start
```

Disable SELinux for httpd & rsyslog:

```shell
setsebool -P httpd_disable_trans 1
setsebool -P syslogd_disable_trans 1
```

## Fedora Installation
*(tested on Fedora 13)*

### Software Installation

Install software requirements:

```shell
yum install -y httpd perl mod_perl mod_ssl mysql mysql-server nscd rsyslog
yum install -y rrdtool htmldoc
```

(required for CPAN)

```shell
yum install -y make perl-CPAN
```

Install Perl modules requirements:

```shell
yum install -y perl-Cache-Cache perl-Crypt-PasswdMD5 perl-Date-Manip
yum install -y perl-DBD-MySQL perl-DBI
yum install -y perl-JSON perl-Linux-Inotify2 perl-List-MoreUtils perl-Locale-Maketext-Lexicon perl-Locale-Maketext-Simple perl-Mail-Sender
yum install -y perl-LDAP perl-Net-SCP perl-Net-Telnet perl-Net-XMPP perl-Proc-PID-File perl-Proc-ProcessTable
yum install -y perl-Readonly-XS perl-Regexp-Assemble perl-Sys-CPU perl-Unix-Syslog perl-Term-ProgressBar perl-URI perl-version perl-XML-Simple

cpan Apache::ASP App::Info DateTime::Format::Strptime LWP Net::FTP Time::Piece
```

Get the latest octopussy source package [here](http://sourceforge.net/project/showfiles.php?group_id=154314).
Launch installation script:

```
tar xvfz octopussy-<version>.tar.gz
cd octopussy
bash LINUX/INSTALL.sh
```

### Fedora Configuration

Change iptables configuration (in `/etc/sysconfig/iptables`) and add this:

```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8888 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 514 -j ACCEPT
-A INPUT -m udp -p udp --dport 514 -j ACCEPT
```

Restart iptables with the new configuration:

```shell
/etc/init.d/iptables restart
```

Disable syslog and enable rsyslog and octopussy at boot time:

```shell
chkconfig --del syslog
chkconfig --add octopussy
chkconfig --add rsyslog
chkconfig --level 2345 octopussy on
chkconfig --level 2345 rsyslog on
```

Modify Rsyslog default configuration (in `/etc/sysconfig/rsyslog`):

```
SYSLOGD_OPTIONS="-c 3"
```

Modify Rsyslog configuration (in `/etc/rsyslog.conf`):

```
	#################
	#### MODULES ####
	#################

	$ModLoad imuxsock.so # provides support for local system logging
	$ModLoad imklog.so   # provides kernel logging support (previously done by rklogd)
	#$ModLoad immark  # provides --MARK-- message capability

	# provides UDP syslog reception
	$ModLoad imudp.so
	$UDPServerRun 514

	# provides TCP syslog reception
	$ModLoad imtcp.so
	$InputTCPServerRun 514


	###########################
	#### GLOBAL DIRECTIVES ####
	###########################

	#
	# Use traditional timestamp format.
	# To enable high precision timestamps, comment out the following line.
	#
	#$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

	#
	# Set the default permissions for all log files.
	#
	$FileOwner root
	$FileGroup adm
	$FileCreateMode 0640
	$DirCreateMode 0755

	#
	# Include all config files in /etc/rsyslog.d/
	#
	$IncludeConfig /etc/rsyslog.d/*.conf
```

And put your server name in `/etc/rsyslog.d/octopussy.conf`:

```
:hostname, !isequal, "your_server_name" ~
```

Restart rsyslog:

```shell
/etc/init.d/rsyslog restart
```

Disable SELinux (didn't find how to disable only for httpd & rsyslog):

Edit `/etc/sysconfig/selinux`:

```
SELINUX=disabled
```

You need to reboot in order to disable SELinux.


## Gentoo Installation

You can take a look at this [documentation](http://en.gentoo-wiki.com/wiki/Octopussy|documentation).


## Standard Installation

### Software Requirements

Before installing Octopussy, be sure to have installed:

  * [Apache2](http://httpd.apache.org)
  * [Apache::ASP](http://www.apache-asp.org)
  * [Perl](http://www.perl.org)
  * [Mod_Perl](http://perl.apache.org)
  * [MySQL](http://mysql.com) or [MariaDB](https://mariadb.org/)
  * nscd
  * [RRDTool](http://oss.oetiker.ch/rrdtool/) & librrd0
  * [RSyslog](http://www.rsyslog.com)
  * [htmldoc](http://www.htmldoc.org)
  * sudo


and these Perl modules (from CPAN):

  * [App::Info](https://metacpan.org/pod/App::Info)
  * [Authen::SASL](https://metacpan.org/pod/Authen::SASL)
  * [Cache::Cache](https://metacpan.org/pod/Cache::Cache)
  * [Crypt::PasswdMD5](https://metacpan.org/pod/Crypt::PasswdMD5)
  * [Data::GUID](https://metacpan.org/pod/Data::GUID)
  * [Date::Manip](https://metacpan.org/pod/Date::Manip)
  * [DateTime::Format::Strptime](https://metacpan.org/pod/DateTime::Format::Strptime)
  * [DBI](https://metacpan.org/pod/DBI) (with [DBD::mysql](https://metacpan.org/pod/DBD::mysql))
  * [Email::MIME](https://metacpan.org/pod/Email::MIME)
  * [Email::Sender](https://metacpan.org/pod/Email::Sender)
  * [Getopt::Long](https://metacpan.org/pod/Getopt::Long)
  * [IO::Socket::SSL](https://metacpan.org/pod/IO::Socket::SSL)
  * [JSON](https://metacpan.org/pod/JSON)
  * [Linux::Inotify2](https://metacpan.org/pod/Linux::Inotify2)
  * [List::MoreUtils](https://metacpan.org/pod/List::MoreUtils)
  * [Locale::Maketext::Lexicon](https://metacpan.org/pod/Locale::Maketext::Lexicon)
  * [Locale::Maketext::Simple](https://metacpan.org/pod/Locale::Maketext::Simple)
  * [LWP](https://metacpan.org/pod/LWP)
  * [Net::FTP](https://metacpan.org/pod/Net::FTP)
  * [Net::LDAP](https://metacpan.org/pod/Net::LDAP)
  * [Net::SCP](https://metacpan.org/pod/Net::SCP)
  * [Net::SSLeay](https://metacpan.org/pod/Net::SSLeay)
  * [Net::Telnet](https://metacpan.org/pod/Net::Telnet)
  * [Net::XMPP](https://metacpan.org/pod/Net::XMPP)
  * [Pod::Find](https://metacpan.org/pod/Pod::Find)
  * [Pod::Usage](https://metacpan.org/pod/Pod::Usage)
  * [Proc::PID::File](https://metacpan.org/pod/Proc::PID::File)
  * [Proc::ProcessTable](https://metacpan.org/pod/Proc::ProcessTable)
  * [Readonly](https://metacpan.org/pod/Readonly)
  * [Regexp::Assemble](https://metacpan.org/pod/Regexp::Assemble)
  * [Sys::CPU](https://metacpan.org/pod/Sys::CPU)
  * [Term::ProgressBar](https://metacpan.org/pod/Term::ProgressBar)
  * [Time::Hires](https://metacpan.org/pod/Time::Hires)
  * [Time::Piece](https://metacpan.org/pod/Time::Piece)
  * [Unix::Syslog](https://metacpan.org/pod/Unix::Syslog)
  * [URI](https://metacpan.org/pod/URI)
  * [version](https://metacpan.org/pod/version)
  * [XML::Simple](https://metacpan.org/pod/XML::Simple) (Take a look [here](http://search.cpan.org/~grantm/XML-Simple-2.18/lib/XML/Simple.pm#ENVIRONMENT) for performance issues)

Install Perl modules requirements:

```shell
cpan Apache::ASP App::Info App::Info::HTTPD Authen::SASL Cache::Cache \
  Crypt::PasswdMD5 Data::GUID Date::Manip \
  Time::Hires Test2::Require::Module \
  DateTime::Format::Strptime DBD::mysql DBI \
  Email::MIME Email::Sender Getopt::Long IO::Socket::SSL \
  JSON Linux::Inotify2 List::MoreUtils \
  Locale::Maketext::Lexicon Locale::Maketext::Simple \
  LWP Net::FTP Net::LDAP Net::SCP Net::SSLeay Net::Telnet Net::XMPP \
  Pod::Find Pod::Usage Proc::PID::File Proc::ProcessTable Readonly \
  Regexp::Assemble Sys::CPU Term::ProgressBar Time::Piece \
  Unix::Syslog URI version XML::Simple
```

### Software Installation

Get the latest octopussy source package [here](http://sourceforge.net/project/showfiles.php?group_id=154314).
If the [LINUX/INSTALL.sh](https://github.com/sebthebert/Octopussy/blob/master/LINUX/INSTALL.sh) installation script didn't work, follow these steps:

#### Create Octopussy user and group

```shell
/usr/sbin/adduser --system --disabled-password --no-create-home --group --quiet octopussy
```

#### Create directories

```shell
/bin/mkdir -p /etc/aat/
/bin/mkdir -p /etc/octopussy/
/bin/mkdir -p /usr/share/aat/
/bin/mkdir -p /usr/share/octopussy/
/bin/mkdir -p /usr/share/perl5/AAT/
/bin/mkdir -p /usr/share/perl5/Octopussy/
/bin/mkdir -p /var/cache/octopussy/asp/
/bin/mkdir -p /var/lib/octopussy/
/bin/mkdir -p /var/run/aat/
/bin/mkdir -p /var/run/octopussy/
```

#### Copy Octopussy files to final destination and change permissions

```shell
/bin/cp -f -r etc/* /etc/
/bin/cp -f -r bin/* /usr/sbin/
/bin/cp -f -r usr/share/aat/* /usr/share/aat/
/bin/cp -f -r usr/share/octopussy/* /usr/share/octopussy/
/bin/cp -f -r lib/AAT* lib/Octo* /usr/share/perl5/
/bin/cp -f -r var/lib/octopussy/* /var/lib/octopussy/
/bin/chown -R octopussy:octopussy /etc/octopussy/ /usr/share/octopussy/ /usr/sbin/octo*
/bin/chown -R octopussy:octopussy /var/cache/octopussy/ /var/lib/octopussy/ /var/run/aat/ /var/run/octopussy/
/bin/chmod 755 /usr/sbin/octo*
```

#### Create a Symlink AAT in `/usr/share/octopussy/` linking to `/usr/share/aat/`

```shell
/bin/ln -f -s /usr/share/aat /usr/share/octopussy/AAT
```

#### Create Octopussy MySQL Database and Table

For MySQL < 5.7:
```shell
/usr/bin/mysql -u root -p < LINUX/OCTOPUSSY.sql
```

For MySQL >= 5.7:
```shell
/usr/bin/mysql -u root -p < LINUX/OCTOPUSSY.mysql57
```

#### Add octo_logrotate to cron.daily

Edit `/etc/cron.daily/octo_logrotate` file and put those lines in it:

```
#!/bin/sh

test -x /usr/sbin/octo_logrotate || exit 0
sudo -u octopussy /usr/sbin/octo_logrotate
```

#### Create init file and make it start at boot

```shell
/bin/ln -f -s /usr/sbin/octopussy /etc/init.d/octopussy
```

In Red Hat like Linux distributions:

```shell
/sbin/chkconfig --add octopussy
```

In Gentoo like Linux distributions:

```shell
rc-update --add octopussy default
```

#### Configure RSyslog for Octopussy

Create the FIFO file for the communication between RSyslog & octo_dispatcher:
(RSyslog writes logs to that file & octo_dispatcher reads logs from that file)

```shell
/bin/mkdir -p /var/spool/octopussy/
/usr/bin/mkfifo /var/spool/octopussy/octo_fifo
/bin/chown -R octopussy:octopussy /var/spool/octopussy/
```

Modify your Rsyslog configuration:

```
	#### Enable TCP & UDP reception

	# provides UDP syslog reception
	$ModLoad imudp
	$UDPServerRun 514

	# provides TCP syslog reception
	$ModLoad imtcp
	$InputTCPServerRun 514


	#### Disable old syslog datetime format

	#$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
```

Then restart your RSyslog daemon

```shell
/etc/init.d/rsyslog restart
```

#### Configure Apache2 for Octopussy

The Octopussy Apache2 configuration is in the `/etc/octopussy/apache2.conf` file.
Here is what you have in this file:

```
	ServerRoot "/etc/octopussy"
	ServerName octopussy

	LockFile /var/lock/apache2/accept-octopussy.lock
	PidFile /var/run/octopussy/apache2.pid

	Timeout 300
	KeepAlive On
	MaxKeepAliveRequests 100
	KeepAliveTimeout 15

	<IfModule mpm_prefork_module>
    	StartServers          5
    	MinSpareServers       5
    	MaxSpareServers      10
    	MaxClients          150
    	MaxRequestsPerChild   0
	</IfModule>

	<IfModule mpm_worker_module>
    	StartServers          2
    	MaxClients          150
    	MinSpareThreads      25
    	MaxSpareThreads      75
    	ThreadsPerChild      25
    	MaxRequestsPerChild   0
	</IfModule>

	User octopussy
	Group octopussy

	DefaultType text/plain

	HostnameLookups Off

	ErrorLog /var/log/apache2/octopussy-error.log
	LogLevel warn

	# Include module configuration:
	Include /etc/apache2/mods-enabled/dir.load
	Include /etc/apache2/mods-enabled/mime.load
	Include /etc/apache2/mods-enabled/perl.load
	Include /etc/apache2/mods-enabled/setenvif.load
	Include /etc/apache2/mods-enabled/ssl.load
	Include /etc/apache2/mods-enabled/dir.conf
	Include /etc/apache2/mods-enabled/mime.conf
	Include /etc/apache2/mods-enabled/setenvif.conf
	Include /etc/apache2/mods-enabled/ssl.conf

	Listen 8888

	LogFormat "%v:%p %h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
	LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
	LogFormat "%h %l %u %t \"%r\" %>s %b" common
	LogFormat "%{Referer}i -> %U" referer
	LogFormat "%{User-agent}i" agent
	CustomLog /var/log/apache2/octopussy.log vhost_combined

	ServerTokens Full
	ServerSignature Off

	<IfModule mod_setenvif.c>
    	BrowserMatch "Mozilla/2" nokeepalive
    	BrowserMatch "MSIE 4\.0b2;" nokeepalive downgrade-1.0 force-response-1.0
    	BrowserMatch "RealPlayer 4\.0" force-response-1.0
    	BrowserMatch "Java/1\.0" force-response-1.0
    	BrowserMatch "JDK/1\.0" force-response-1.0
    	BrowserMatch "Microsoft Data Access Internet Publishing Provider" redirect-carefully
    	BrowserMatch "MS FrontPage" redirect-carefully
    	BrowserMatch "^WebDrive" redirect-carefully
    	BrowserMatch "^WebDAVFS/1.[012]" redirect-carefully
    	BrowserMatch "^gnome-vfs/1.0" redirect-carefully
    	BrowserMatch "^XML Spy" redirect-carefully
    	BrowserMatch "^Dreamweaver-WebDAV-SCM1" redirect-carefully
	</IfModule>

	ServerName octopussy
  	DocumentRoot /usr/share/octopussy/
  	SSLEngine on
	# SSLCACertificateFile /etc/octopussy/CA/cacert.pem
	# SSLCARevocationFile /etc/octopussy/CA/cacert.crl
  	SSLCertificateFile    /etc/octopussy/server.crt
  	SSLCertificateKeyFile /etc/octopussy/server.key
  	DirectoryIndex index.asp

  	<Directory "/usr/share/octopussy/">
    	Options +FollowSymLinks
  	</Directory>

  	PerlModule Bundle::Apache2
  	PerlModule Apache2::compat

  	<Files ~ (\.asp)>
    	AddDefaultCharset utf-8
    	SetHandler perl-script
    	PerlHandler Apache::ASP
    	PerlSetVar StateDB MLDBM::Sync::SDBM_File
    	PerlSetVar Global /usr/share/octopussy
    	PerlSetVar StateDir /var/cache/octopussy/asp
    	PerlSetVar RequestParams 1
    	PerlSetVar XMLSubsMatch \w+:[\w\-]+
  	</Files>
```

If you've got a problem with Apache::ASP configuration, read the documentation [here](http://www.apache-asp.org/config.html).

You also need to enable `mod_dir` & `mod_ssl`.

```shell
a2enmod dir
a2enmod ssl
```

Generate self-signed Certificate for Octopussy Web Server:

```
openssl genrsa > /etc/octopussy/server.key
openssl req -new -x509 -nodes -sha1 -days 365 -key /etc/octopussy/server.key > /etc/octopussy/server.crt
```

The `/etc/init.d/octopussy web-start` command launches Apache2 like this:

```shell
/usr/sbin/apache2 -f /etc/octopussy/apache2.conf -k start
```

#### Start Octopussy

```shell
/etc/init.d/octopussy start
```

If Octopussy doesn't work at this stage and you don't know why, submit a [Bug Report](https://github.com/sebthebert/Octopussy/issues) !


# Use PostgreSQL instead of MySQL

**WORK IN PROGRESS...**

If you want to use [PostgreSQL](http://www.postgresql.org/) instead of [MySQL](http://www.mysql.com/), you need to make this changes:
  * Install [DBD::Pg](https://metacpan.org/pod/DBD::Pg) Perl module
  * Create your Octopussy PostgreSQL user
```sql
postgres=# CREATE USER octopussy WITH PASSWORD 'octopussy';
```
  * Create your Octopussy PostgreSQL database:
```sql
postgres=# CREATE DATABASE octopussy OWNER octopussy;
```
  * Create your Octopussy **\_alerts\_** table
```sql
CREATE TABLE _alerts_ (log_id SERIAL, alert_id varchar(250) default NULL,
status varchar(50) default 'Opened', level varchar(50) default NULL,
date_time timestamp default NULL, device varchar(250) default NULL,
log text default NULL, comment text default NULL, PRIMARY KEY  (log_id));
```

# Configuring non-root user(s) to launch Octopussy programs

If you want to launch Octopussy programs with non-root user(s), you need to configure sudo.
Edit sudoers file with:

```shell
visudo
```

Then add these lines:

```
User_Alias OCTO_USERS = username1,username2

OCTO_USERS ALL=(root) NOPASSWD: /usr/sbin/octopussy
OCTO_USERS ALL=(octopussy) NOPASSWD: /usr/sbin/octo_*
```

Then `username1` & `username2` will be able to lauch these kind of commands:

```
sudo -u root /usr/sbin/octopussy restart
sudo -u octopussy /usr/sbin/octo_logrotate
sudo -u octopussy /usr/sbin/octo_reporter ...
```
