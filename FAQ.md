FAQ 
===

### How can I change the Logs Wizard messages limit ?

You can change that limit by editing the parameter *wizard_max_msgs* in */etc/aat/aat.xml* file.

### How can I configure Octopussy to interact with my LDAP/SMTP/Jabber server ? 

You can configure your LDAP/SMTP/Jabber server on the WebUI [[web#system configuration page|System Configuration Page]].

### How can I contribute to Octopussy Project ? 

Look at the Contribution section of the [Community](/community) Page.

### How can I create a new Octopussy Plugin ?

Look at the [Plugin Howto](/documentation/howtos/plugin) Page.

### How can I create a new Octopussy Service ? 

Look at the [New Service Creation](/documentation/tutorials/new_service) tutorial.

### How can I handle Windows Hosts ?

You can use [Snare Agent for Windows](http://www.intersectalliance.com/projects/SnareWindows/) to send Windows logs to Octopussy.

Take a look at [Administrator Guide/Installation/Windows Host with Snare Agent](/documentation/guides/administrator_guide/01_installation).

### How can I login to Octopussy Web interface ?

The default **login / password** are **admin / admin**.

### How can I see how many logs I received ?

You can see the number of syslog messages you received on the last minute on the WebUI [[web#main page|Main Page]].

### What are software requirements for Octopussy ?

You can get the list of software requirements in [Administrator Guide/Installation/Standard Installation/Software Requirements](/documentation/guides/administrator_guide/01_installation).

### What is a Device in Octopussy ?

A Device is one source of syslog messages.
A Device can contains one or more Services.

### What is a Message in Octopussy ?

A Message is an object with 6 fields: 

  * *message id*: this id is unique.
  * *pattern*: this pattern is converted to regexp by **Octopussy Parser** to match syslog messages.
  * *logelevel*: this level represents the criticity of the Message.
  * *taxonomy*: it defines the category of the Message. (Auth, Network, System...)
  * *table*: this table is used by **Octopussy Reporter** to generate Reports. 
  * *rank*: this is the position of this Message in its Service.

### What is a Plugin in Octopussy ?

Look at the [Plugin Howto](/documentation/howtos/plugin) Page.

### What is a Service in Octopussy ?

A Service is a collection of Messages.