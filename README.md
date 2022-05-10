**** SET UP ****
- Choose 7.45GB
- 4.2 GB partition
- 500 MB swap
- In network tab in virtual box, change from NAT to bridged.

* Neccessary installs
	- update package:
		apt-get update && apt-get upgrade
	- install sudo:
		apt-get install sudo
	- install vim: (if not enough memory, uninstall this)
		apt-get install vim
	- install parted: (if not enough memory, uninstall this)
		apt-get install parted

* Commonly used command-line
	- reboot:
		reboot
	- check memory
		parted -> unit GB -> print all
	- find IP hostname
		hostname -I
		ip addr
	- find default gateway
		ip route show


* You must create a non-root user to connect to the machine and work.
* Use sudo, with this user, to be able to perform operation requiring special rights
	- In root:
		+ create user
			adduser [username]
		+ give user sudo right
			usermod -aG sudo [username]
		+ switching between user
			su - [username]

* We don’t want you to use the DHCP service of your machine. You’ve got to configure it to have a static IP and a Netmask in \30.
	- \30 means the first 30 bits -> the first 3 octets remains the same. +/-1 in the last octet could be a safe choice.
	- ping the wanted IP address. If there is not signal => could be free!
	- making changes in /etc/network/interfaces
		+ sudo vim /etc/network/interfaces
		+ changes e.g:
			iface enp0s3 inet static
				address 192.168.1.129
				netmask 255.255.255.252
				gateway 192.168.1.1

* You have to change the default port of the SSH service by the one of your choice. SSH access HAS TO be done with publickeys. SSH root access SHOULD NOT be allowed directly, but with a user who can be root.
	- To copy public key to VM machine, in local:
		+ have a ready public key, if not, generate one
			ssh-keygen-o
		+ copy public key to VM. This could be later checked in VM at ~/.ssh/authorized_keys
			ssh-copy-id [VM_username]@[VM_IP]
	- To change port, make changes in /etc/ssh/sshd_config
		+ sudo vim /etc/ssh/sshd_config
		+ changes e.g:
			Port 2608
	- To forbid directly access to root, make changes in /etc/ssh/sshd_config
		+ sudo vim /etc/ssh/sshd_config
		+ changes e.g:
			PermitRootLogin no
	- NOTE! from this moment onward, cannot access root directly and access to any user must be with correct port.
		+ ssh [VM_username]@[VM_IP] -p [PORT_NUMBER]
		
* You have to set the rules of your firewall on your server only with the services used outside the VM
	- UFW (uncomplicated firewall)
		+ install
			sudo apt install ufw
		+ status of  UFW:
			sudo ufw status verbose
		+ check rule status:
			sudo ufw status numbered
		+ delete rule:
			sudo ufw delete [RULE_NUMBER]
	- for SSH service:
		+ allow the specific ports
			sudo ufw allow [PORT_NUMBER]
		
