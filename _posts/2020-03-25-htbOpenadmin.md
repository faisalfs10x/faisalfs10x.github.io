---
layout: post
title: "OpenAdmin write-up"
date: 2020-03-25 00:22:00 -0000
categories: jekyll
permalink: htbOpenAdmin
exploit: RCE

---

# HTB - OpenAdmin

![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/intro.png "openadmin intro")
### Recon
    nmap -Pn -sV --script vulners --script-args mincvss=7.0 -p22,80 10.10.10.171 
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/0.png)

From the nmap scan, there are 2 services, OpenSSH 7.6p1 and Apache httpd 2.4.29.

As usual, we started with dirsearch to brute force site structure including directories and files in websites and we found interesting directory that is `/music`.
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/1.png)
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/2.png)

However, i found nothing in the `/music` directory until i clicked the login button and then found this page.
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/2.1.png)

On this page, it seem that it has old version of OpenNetAdmin 18.1.1. 
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/3.png)

Then, we came to searchsploit for the version and got `RCE` bash script.
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/4.png)

### Exploit
First, we have to convert the bash script using `dos2unix` and feed the script with URL argument and we got the `www-data shell`.
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/5.png)

#### Discover interesting database configuration files that lead to jimmy shell:
In the box, we do some enumeration until we found interesting database configuration file in **/opt/ona/www/local/config** and we get mysql database config file. Noted that for the `db_passwd`. It seems the password is for database but it might be useful for the users. We also discovered some users on the box that are `jimmy` and `joanna`
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/6.png)
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/7.png)

I decided to try those credential and got a valid credential for jimmy that is **jimmy:n1nj4W4rri0R!** and `su` into jimmy. Hmmm, password reuse again. Then, enumerate again until found `main.php` file. What is it??
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/8.png)

It seem like php session that parsing system command and we tried to curl the file and we got id_rsa for joanna :XD. The localhost server was serving on port 52846 that we obtain using `netstat -alnp` command.
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/9.png)

#### Cracking id_rsa and recover the key to gain joanna shell:
Again, copied the id_rsa content to local machine into `hash.txt` and tried to crack the key using `john` and we got **bloodninjas** as the key. Yuhuuuu!!
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/10.png)

Now, lets SSH into joanna and grab user flag..

![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/11.png)

### Privilege escalation

To escalate we tried basic enum using `sudo -l` and found that joanna can run `/bin/nano /opt/priv` without any password.
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/12.png)

Referring at [GTFOBins](https://gtfobins.github.io/gtfobins/nano/#shell), there’s a way to execute command which inside nano text editor to escape to root shell.
![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/13.png)

Then, we can read the root.txt flag on the box after getting privilege as root. XD.

![alt text](https://github.com/faisalfs10x/faisalfs10x.github.io/blob/master/asset/htbwriteup/linux/openadmin/14.png)

Thanks.
