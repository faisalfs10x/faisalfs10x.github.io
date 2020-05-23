---
layout: post
title: "Postman write-up"
date: 2020-03-22 00:22:00 -0000
categories: jekyll
permalink: htbPostman
exploit: Redis 4.0.9 | MiniServ 1.910 (Webmin httpd)
tags: [redis, webmin]

---

# HTB - Postman

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/intro.PNG "postman intro")

### Recon
    nmap -Pn --open -sC -sV -p- -T4 10.10.10.160 

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/1.png)

From the nmap scan, I discovered uncommon ports that are 6379 and 10000, Redis key-value store 4.0.9 and http MiniServ 1.910 (Webmin httpd) respectively.

### Exploit
Then, I found [Redis RCE exploit](https://packetstormsecurity.com/files/134200/Redis-Remote-Command-Execution.html) from Packet Storm Security. We could exploit unauthenticated Redis server by writing a content inside the memory of Redis server. We have to create our own SSH keys and insert the public key inside the Redis server to be able SSH into the box.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/2.png)

#### Writing the Public Key into Memory using redis-CLI:

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/3.png)


#### Redis user to Matt:

After that we can SSH into redis user on the box. However, we could not read `user.txt` yet. We need to escalate to `Matt` user first.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/4.png)

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/5.png)


While doing enumeration, we found `id_rsa.bak` file in `/opt` that is an id_rsa backup for user Matt.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/6.png)


Then, we copied the `id_rsa` content into our local machine to crack the key.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/7.png)


#### Cracking id_rsa key:

We have to convert the format using john utility called `ssh2john` first before cracking the key. ssh2john can converts the private key to a format that john can crack it.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/8.png)


We are able to crack the key and got `computer2008` as the key.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/9.png)


Then, we tried to SSH as user Matt but the connection was closed by the server.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/10.png)


However, we could escalate into Matt by substitute user and read the `user.txt` flag.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/11.png)


### Privilege escalation
Based on nmap scan, we found that `Webmin 1.910 service on port 10000` was up. By using searchsploit, we found RCE exploit in Metasploit module.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/12.png)


We filled all the required setting for the module using same credential for Matt user and got root shell.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/13.png)

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/postman/14.png)
