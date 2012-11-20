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


Take snapshot.



Setup Vagrant
-------------
Login as root.

Install basic packages:

	# yum install -y man nano sudo wget

Create `vagrant` user:

	# useradd vagrant
	# passwd vagrant (vagrant)

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


Install Ruby (source)
---------------------
Install required packages:

	$ sudo yum install -y openssl-devel readline-devel zlib-devel

Create `~/src` directory:

	$ mkdir -p ~/src

Install `libyaml` from source:

	$ cd ~/src
	$ wget http://pyyaml.org/download/libyaml/yaml-0.1.4.tar.gz
	$ tar zxvf yaml-0.1.4.tar.gz
	$ cd yaml-0.1.4
	$ ./configure --prefix=/usr/local
	$ make && sudo make install
	
Install Ruby from source:

	$ cd ~/src
	$ wget http://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.3-p327.tar.gz
	$ tar zxvf ruby-1.9.3-p327.tar.gz
	$ cd ruby-1.9.3-p327
	$ ./configure --prefix=/opt/vagrant_ruby --enable-shared --disable-install-doc --with-opt-dir=/usr/local
	$ make && sudo make install

Install Rubygems from source:

	$ cd ~/src
	$ wget http://production.cf.rubygems.org/rubygems/rubygems-1.8.24.tgz
	$ tar zxvf rubygems-1.8.24.tgz
	$ cd rubygems-1.8.24
	$ sudo /opt/vagrant_ruby/bin/ruby setup.rb

Create `/etc/gemrc` defaults:

```
$ sudo sh -c "echo 'install: --no-rdoc --no-ri
update:  --no-rdoc --no-ri' > /etc/gemrc"
```

Install Chef gem:

	$ sudo /opt/vagrant_ruby/bin/gem install chef --no-rdoc --no-ri

Take snapshot.


Edit Path
---------
Add paths to `/etc/profile.d`:

	$ sudo sh -c "echo 'pathmunge /sbin' > /etc/profile.d/path_sbin.sh"
	$ sudo sh -c "echo 'pathmunge /usr/sbin' > /etc/profile.d/path_usr_sbin.sh"
	$ sudo sh -c "echo 'pathmunge /opt/vagrant_ruby/bin' > /etc/profile.d/path_opt_vagrant_ruby_bin.sh"
	$ source /etc/profile

Take snapshot.


Clean Up
--------
Disable `kudzu`:

	$ sudo chkconfig kudzu off

Remove packages and empty caches:

    $ rm -rf ~/src
    $ sudo yum remove -y *-devel
    $ sudo yum clean all
    $ sudo rm /root/install.log

Take snapshot.


Package
-------
CD to directory containing virtual machine disk and package box.

	$ vagrant package --base centos-5.8-minimal
	$ vagrant box add centos-58-x86_64-minimal ./package.box
	$ ls -alh package.box
	-rw-r--r--  1 doc  staff   468M Nov 20 08:05 package.box
