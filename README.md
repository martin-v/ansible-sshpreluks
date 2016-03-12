sshpreluks
==========

Configure initrd to allow remote unlock of full disk encrypted systems over ssh.


This module is only manual tested on debian jessie, but it should work on other
actual debian installations.


Requirements
------------

The role installs on host:

  * dropbear
  * initramfs-tools
  * cryptsetup


This role make only sense if the target's root partion is encrypted.


Role Variables
--------------

### Required Variables:

#### sshpreluks_pubkeys

This variable must contain the public ssh keys of all user who should able to unlock the encrypted root partition. The user names in the variable are only for
documentation.

**!!! Attention all of this user will have root access to the system !!!**

	sshpreluks_pubkeys:
	  root:
	  - ssh-rsa AAAAB3NzaC[...]AXNqw== admin@remote
	  - ssh-rsa AAAAB3NzaC[...]AXNqw== admin@notebook
	  sysadmin:
	  - ssh-rsa AAAAB3NzaC[...]AXNqw== user_with_disk_crypto_pw@remote


#### sshpreluks_network_config

Specify the network ip configuration. This variable is mandatory. The structure for this variable is:

	<Host-IP>::<Gateway-IP>:<Netmask>:<Hostname>:<Device>:<autoconf>

For syntax details take a look at [kernel nfsroot documentation](
https://www.kernel.org/doc/Documentation/filesystems/nfs/nfsroot.txt).

Example for dhcp:

	sshpreluks_network_config: ":::::eth0:dhcp"

Example for static ip:

	sshpreluks_network_config: "192.168.1.10::192.168.1.1:255.255.255.0:myservername:eth0:off"


### Optional Variables:

#### sshpreluks_network_device

Specify the device for for the pre luks network connection. Defaults to `eth0`.

	sshpreluks_network_interface: eth1



Recommendations
---------------

The dropbear ssh server will use a other ssh server key as your normal ssh server, so you
should use a separate `KnownHostsFile`.

To unlock a remote server you can use for example on your local machine the following function:

	function sshpreluks() {
		HOSTNAME=$1
		ssh -o "UserKnownHostsFile=~/.ssh/known_hosts.initramfs" \
			root@$HOSTNAME \
			"/lib/cryptsetup/askpass \"Enter luks password for $HOSTNAME:\" > /lib/cryptsetup/passfifo"
	}


Dependencies
------------

None.


Example Playbook
----------------

	---

	- hosts: all
	  remote_user: root
	  vars_files:
		- sshpreluks_vars.yml
	  roles:
		- martin-v.sshpreluks



Example variables file
----------------------

	---

	sshpreluks_network: ":::::eth0:dhcp"

	sshpreluks_pubkeys:
	  root:
	  - ssh-rsa AAAAB3NzaC[...]AXNqw== admin@workstation


Open tasks
----------

0. Find solution for automatic functional tests and implement it
0. Test on other debian based systems


License
-------

MIT

Author Information
------------------

This role was created in 2016 by [Martin V](https://github.com/martin-v).
