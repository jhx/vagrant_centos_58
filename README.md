Vagrant CentOS 5.8 setup
========================

This document describes how to setup a CentOS minimal installation using vagrant.


Requirements
------------
- [VirtualBox](https://www.virtualbox.org)
- [Vagrant](http://vagrantup.com)
- [CentOS 5.8](http://mirror.raystedman.net/centos/5.8/isos/x86_64/)


Configure VirtualBox
--------------------
General:

- Name: **`centos-5.8-minimal`**
- Type: **Linux**
- Version: **Red Hat (64 bit)**

System:

- Motherboard
	- Base Memory: **360 MB**
	- Boot Order: **CD/DVD-ROM, Hard Disk**
	- Extended Features: **Enable IO APIC, Hardware clock in UTC time**

Display:

- *(use default settings)*

Storage:

- Controller: SATA **`centos-5.8-minimal.vdi`**
- Virtual Size: **64.00 GB**
- Details: **Dynamically allocated**

Audio:

- **Disable audio**

Network:

- Adapter 1:
	- **Enable Network Adapter**
	- Attached to **NAT**
	- Port Forwarding
		- Name: **ssh**
		- Protocol: **TCP**
		- Host IP: **127.0.0.1**
		- Host Port: **2222**
		- Guest IP: 
		- Guest Port: **22**
		
- Adapters 2..4:
	- **Disable Network Adapter**

Ports:

- Serial Ports: **Disable Serial Port**
- USB: **Disable USB Controller**

Shared Folders:

- *(use default settings)*


Install CentOS 5.8
------------------
Connect CD/DVD Drive to `CentOS-5.8-x86_64-bin-DVD-1of2`

Start VM

- Install in non-GUI mode

		boot: linux text

- CD Found: **Skip**
- Language Selection: **English**
- Keyboard Selection: **us**
- Initialize drive, erasing all data: **Yes**
- Partitioning Type: **Remove linux partitions…sda**
- Remove all Linux partitions: **Yes**
- Review Partition Layout: **No**
- Configure Network Interface: **Yes**

		[*] Activate on boot
		[*] Enable IPv4 support
		[ ] Enable IPv6 support

- IPv4 Configuration for eth0

		(*) Dynamic IP configuration (DHCP)
		( ) Manual address configuration

- Hostname Configuration

		( ) automatically via DHCP
		(*) manually: vagrant-centos-58

- Time Zone Selection

		[*] System clock uses UTC
		
		America/Chicago

- Root Password

		Password: vagrant
		Password (confirm): vagrant

- Package selection

		[ ] Desktop - Gnome
		[ ] Desktop - KDE
		[ ] Server
		[ ] Server - GUI
		[ ] Clustering
		[ ] Storage Clustering
		
		[*] Customize software selection

- Package Group Selection

		[ ] (deselect all package groups)

- Installation to begin: **OK**

- Complete: **Reboot**


Setup Vagrant
-------------
Login as root.

Create `vagrant` user:

	# useradd vagrant
	# passwd vagrant (vagrant)

Install basic packages:

	# yum install -y man nano sudo wget

Configure sudo via `visudo`:

	# visudo

- Disable `requiretty`:

		# Defaults    requiretty

- Add `PATH`, `SSH_AUTH_SOCK` to `env_keep`:

		PATH SSH_AUTH_SOCK

- Enable group `wheel`:

	    %wheel  ALL=(ALL)       ALL

- Enable user `vagrant`:

	    vagrant ALL=(ALL)       NOPASSWD: ALL

- Write file and exit

		<ESC>:w:q

Test login via ssh from separate terminal.
Logout.
Take snapshot.


Install Guest Additions
----------------------------------
Devices > Install Guest Additions…

Login as vagrant.

Install packages required for guest additions:

	$ sudo yum install -y which bzip2 gcc make kernel-devel-`uname -r`

Mount CDROM & run installation script (ignore error related to Window System drivers):

    $ sudo mount /dev/cdrom /media
    $ sudo sh /media/VBoxLinuxAdditions.run

Remove unneeded packages:

	$ sudo yum remove -y kernel-devel

Take snapshot.


Configure SSH
-------------
Install packages required for ssh:

	$ sudo yum install -y curl

Configure `~/.ssh`:

    $ mkdir -m 0700 ~/.ssh && cd ~/.ssh
    $ curl -k https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub > authorized_keys
    $ chmod 0644 authorized_keys

Configure `/etc/ssh/sshd_config`:

    $ sudo nano /etc/ssh/sshd_config
   	...
    UseDNS no
    ...

Restart `sshd` daemon:

    $ sudo /sbin/service sshd restart

Take snapshot.


Install Additional Repos
------------------------
Download and install EPEL:

    $ sudo rpm -Uvh http://dl.fedoraproject.org/pub/epel/5/x86_64/epel-release-5-4.noarch.rpm

Download and install RBEL:

	$ sudo rpm -Uvh http://rbel.frameos.org/stable/el5/x86_64/rbel5-release-1.0-2.el5.noarch.rpm

Take snapshot.


Install Ruby
------------
Install Ruby and related packages:

	$ sudo yum install -y ruby.x86_64 rubygems

Create `/etc/gemrc` defaults:

	$ sudo nano /etc/gemrc

	install: --no-rdoc --no-ri 
	update:  --no-rdoc --no-ri

Install Chef gem:

	$ sudo gem install chef

Take snapshot.


Edit Path
---------
Add paths to `/etc/profile.d`:

	$ sudo sh -c "echo 'pathmunge /sbin' > /etc/profile.d/path_sbin.sh"
	$ sudo sh -c "echo 'pathmunge /usr/sbin' > /etc/profile.d/path_usr_sbin.sh"

Take snapshot.


Clean Up
--------
Empty caches:

    $ sudo yum clean all

