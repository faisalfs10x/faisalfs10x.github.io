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
tags: [linux, persistence,]

---

This post will cover a few common linux persistence techniques used by adversary to establish permanent access. The adversary is attempting to keep their foothold.
Persistence refers to strategies used by adversaries to maintain access to systems despite restarts, changing credentials, and other disruptions that may terminate their access. 

---
### Creating privilege account

The adversary tends to create a high privilege account to maintain access on the compromised system. They usually create a user with a sudo privilege across the system.

`sudo useradd -p $(openssl passwd -1 $USERPASS) -s /bin/bash -G sudo $USERNAME`
    




