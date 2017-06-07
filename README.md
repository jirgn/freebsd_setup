# FreeBsd as NAS

some resources for info about freebsd

* [Freebsd Handbook](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/)
* [Freebsd Wiki](https://wiki.freebsd.org/)
* [Hukl's toolbox](https://github.com/hukl/freebsd-toolbox)
* [Hukl's videocasts](https://vimeo.com/channels/freebsdguides)

## Installation
### Prepare usb stick
* Download Image from [FTP](ftp://ftp.freebsd.org/pub/FreeBSD/releases/ISO-IMAGES/11.0/FreeBSD-11.0-RELEASE-amd64-memstick.img)
* Follow Instructions from [Manual](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/bsdinstall-pre.html)

## Basic System Setup
Make your main system as small as possible, so you easily can upgrade later on.
All further capabilities should have there own jails and will be ignored here.

### Software installer
There are two ways of installing software in bsd. 
* pkg - packages for binary precompiled software
* ports - src based installation that will be compiled on your system.
In most cases the two ways can coexist, but in some cases, when you for example need header files of dependend packages that are not present, because you used pkg, it will get tricky. So our decision goes for the port system, cause of its greater flexibility and up to date software.

#### Prepare Port-System

For details see [Documentation](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports-using.html)

	// first install
	# portsnap fetch
	# portsnap extract
	
	// update
	# portsnap fetch update

Ports will be installed in /usr/ports. 
All ports are based on 'make' so they can be installed/compiled via 

	make install
	make clean

For en easy flexible control over the 'make' task we use a tool called 'portmaster'
Install it via 

	cd /usr/ports/ports-mgmt/portmaster
	make install
	make clean

portmaster can then be uses via

	portmaster /<portfolder>/<port-name>

### SUDO

Install sudo with

	portmaster security/sudo

add your user to group wheel and make sure this group has sudo access via

	visudo

### SSH

install ssh if not already present
copy your public key into ~/.ssh/authorized_keys on the server or jail

check if the connection via key can be established
delete password access in /etc/ssh/sshd_config by setting

	ChallangeResponseAuthentication no

Make sure root access is disabled

	PermitRootLogin no

## Jails

for simpler jail management we use ezjail

	// install
	portmaster sysutils/ezjail

see http://erdgeist.org/arts/software/ezjail/

### Static Ip Binding and aliases 

Before setting up jails we need some ip addresses for these jails.
This can easily be achieved setting some aliases in /etc/rc.conf like

	// /etc/rc.conf
	# network
	ifconfig_igb0="inet 192.168.178.100 netmask 255.255.255.0"
	defaultrouter="192.168.178.1"

	ifconfig_igb0_alias0="inet 192.168.178.101 netmask 255.255.255.0"
	ifconfig_igb0_alias1="inet 192.168.178.102 netmask 255.255.255.0"
	ifconfig_igb0_alias2="inet 192.168.178.103 netmask 255.255.255.0"

### activate ezjail
also activate ezjail for the system

	// /etc/rc.conf
	# jails
	ezjail_enable="YES"

and also activate zfs support if using zfs

	// /etc/rc.conf
	# ZFS options

	# Setting this to YES will start to manage the basejail and newjail in ZFS
	ezjail_use_zfs="YES"

	# Setting this to YES will manage ALL new jails in their own zfs
	ezjail_use_zfs_for_jails="YES"

	# The name of the ZFS ezjail should create jails on, it will be mounted at the ezjail_jaildir
	ezjail_jailzfs="zroot/ezjail"

### Install Basejail

	sudo ezjail-admin install -msp

### Flavours

are a kind of templates for jails and are located in /usr/jails/flavours

### Create a jail

	sudo ezjail create -f <flavour-name> <host-name> <ip-address>

### Start jail

	sudo ezjail start <host-name>


### Update Ports Tree

	sudo ezjail-admin -P

### Mount Filesystems

each jail has it's own fstab file which is located in 
/etc/fstab.<jailname>
to inject some local filesystems put them in here in the standard fstab form
