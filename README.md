#### **SET UP**
Choose 7.45GB

4.2 GB partition

500 MB swap

In network tab in virtual box, change from NAT to bridged.

---

#### **NECCESSARY INSTALLS**
update package <br>
     `apt-get update && apt-get upgrade`

install sudo <br>
     `apt-get install sudo`

install vim (if not enough memory, uninstall this)	<br>
     `apt-get install vim`

install parted (if not enough memory, uninstall this) <br>
     `apt-get install parted`

---

#### **COMMONLY USED COMMAND LINE**
reboot: <br>
     `reboot`

check memory <br>
     `parted` -> `unit GB` -> `print all`

find IP hostname <br>
     `hostname -I` or `ip addr`

find default gateway<br>
     `ip route show`

---

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
			sudo ufw allow [PORT_NUMBER]/[NET_ID]
		+ to check [NET_ID]
			sudo ss -tunlp
	- to prevent too many attempt to connect to ssh service. ufw will deny IP that try to connect 6 or more times within 30 seconds.
		sudo ufw limit [PORT_NUMBER]/[NET_ID]
	- prevent the most common port
		sudo ufw deny 22/tcp

* You have to set a DOS (Denial Of Service Attack) protection on your open ports of your VM.
	- install fail2ban:
		sudo apt-get install fail2ban
	- check status
		check status of a jail (for e.g sshd)
			sudo fail2ban-client status sshd
		check banned ip
			sudo fail2ban-client banned
	- check log of ban:
		/var/log/fail2ban.log
	- configure for fail2ban (ssh)
		create a file sshd.local in /etc/fail2ban/jail.d/
		edit the file as e.g below:
			[sshd]
			enabled = true
			filter = sshd
			port = 2608
			logpath = %(sshd_log)s
			maxretry = 3
			bantime = 600
	- unban ip
		sudo fail2ban-client set sshd unbanip 192.168.1.126

* You have to set a protection against scans on your VM’s open ports.
	useful links:
		https://akhil.io/blog/custom-fail2ban-filters
		https://serverfault.com/questions/629709/trouble-with-fail2ban-ufw-portscan-filter
		https://phoenixnap.com/kb/nmap-scan-open-ports
	to scan ports:
		sudo nmap -PN [IP_ADDRESS]
	add custom rule in fail2ban
		to check if regex is correct:
			 sudo fail2ban-regex  /var/log/ufw.log '.*\[UFW BLOCK\] .* SRC=<HOST> .* PROTO=TCP ' --print-all-matched
		in filter.d:
			create a custom rule file ([filename].conf) this is to find a matching pattern in a file that will be later declared in rule.
				[Definition]
				failregex = .*\[UFW BLOCK\] .* SRC=<HOST> .* PROTO=TCP
		in local.d:
			create a custom rule file ([filename].local) and set rules for this filter
				[myportban]
				enabled = true
				filter = myportban
				logpath = /var/log/ufw.log
				maxretry = 3
				findtime = 20
				bantime = 120

* Stop the services you don’t need for this project.
	THIS SHOULD BE CHECKED AT SCHOOL!

* Create a script that updates all the sources of package, then your packages and which logs the whole in a file named /var log/update_script.log. Create a scheduled task for this script once a week at 4AM and every time the machine reboots.
