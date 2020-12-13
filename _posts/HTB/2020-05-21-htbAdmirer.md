---
layout: single
title: Exploiting Adminer 4.6.2 File Disclosure Vulnerability & Python Library Hijacking for PrivEsc
excerpt: "Admirer is an easy box that need to exploit Adminer 4.6.2 to get credential for initial shell then abusing shutil module for python library hijacking to escalate into root shell."
date: 2020-05-21
classes: wide
header:
  teaser: /asset/htbwriteup/linux/admirer/intro.PNG
  teaser_home_page: true
  icon: /assets/hackthebox.webp
categories:
  - hackthebox
permalink: /htb/htbAdmirer
tags: [adminer, shutil, python library hijacking]

---

# HTB - Admirer

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/intro.PNG)

Admirer is an easy box that need to exploit Adminer 4.6.2 to get credential for initial shell then abusing shutil module for python library hijacking to escalate into root shell

---
## Recon

- Running nmap scan and we got 3 services up that are ftp,ssh and http. We can't login as anonymous in ftp, so moved to http service.
    
![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/1nmap.png)

- We checked `robots.txt` and found `Disallow: /admin-dir`
- Fire-up gobuster with big wordlist and scan for `php,txt,xml` and `html` extension. Whoa, we got `contacts.txt` and `credentails.txt`.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/2.png))

- Gathered all the information and jot down the creds.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/3.png)

- We have the ftp credential and login via browser and download the `dump.sql ` and `html.tar.gz` files

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/4.png)

- I found database credentials in `index.php` but since the nmap scan didn’t reveal any database service, it might be internal database server.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/5.png)

- In the `html.tar.gz`, we have a few php files under `utility-scripts`. The **admin_tasks.php** has the source for administrative tasks page while **db_admin.php** revealed the credentials for  SQL database. The next web content to discover via brute-forcing may be `utility-scripts`?

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/6.png)

- Reviewing all the php files revealed us to more credentials as below.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/7.png)

- Then, gobuster time again... under `utility-scripts` and we have `adminer.php`. My instinct was right as it is a database file.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/8.png)

- Browsing to `adminer.php` lead us to  Adminer 4.6.2 login. Try to login as waldo with the cred but the connection refused.

### What is Adminer?
- Adminer (formerly phpMinAdmin) is a full-featured database management tool written in PHP. Conversely to phpMyAdmin, it consist of a single file ready to deploy to the target server. Adminer is available for MySQL, MariaDB, PostgreSQL, SQLite, MS SQL, Oracle, Firebird, SimpleDB, Elasticsearch and MongoDD. 
- Adminer is a PHP administration tool which users can host on their web sites to enable them to remotely administer MySQL databases.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/9.png)

---
## Exploit

- Upon searching for Adminer 4.6.2 exploit, we found this [article](https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool) that explain Serious Vulnerability Discovered in Adminer database Administration Tool.

### **How Does It Work?**
- First, the attacker will access the victim’s Adminer instance, but instead of trying to connect to the  
victim’s MySQL database, they connect “back” to their own MySQL database hosted on their own server.
- Second, using the victim’s Adminer (connected to their own database) – they use the MySQL command  ‘LOAD DATA LOCAL’, specifying a local file on the victim’s server. This command is used to load data from a file local to the Adminer instance, into a database.
- Third – The attacker, using the victim’s Adminer, disconnects from his own database and connects to the   victim’s database using the credentials they have just obtained. With access to the database they could read sensitive information, such as customer details. [ Source - [foregenix.com](https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool)]

---

- Essentially I had to install a MySQL server on my computer, construct the database, construct a table with columns and log into my victim's administrator 's database to dump any local file.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/10.png)

- Then, we are able to login into admirer database.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/12.png)

- Next, we must create a table with a few columns and dump the `adminer.php` via load data local infile statement. The `LOAD DATA INFILE` statement allows you to read data from a text file and import the file’s data into a database table very fast.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/14.png)

- After dumping `adminer.php`, we could retrieved waldo password that is **<h5b~yK3F#{PaPB&dA}{H>**

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/15.png)

- Then, we can connect to SSH using waldo credential and get the user flag.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/16(user).png)

---
## Privilege escalation

- I used **sudo -l** command to see what waldo could do without root access.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/17.png)

- Analysing `/opt/scripts/admin_tasks.sh` revealed us a function name **backup_web()**, It calls python script in same directory.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/18.png)

- Trying to run `sudo /opt/scripts/backup.py` give us the idea to enum the script. It is running successfully.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/19.png)

- We can achieve root access if we modify the Bash or Python script. However, both files aren't user-writable.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/20.png)

- By investigating `backup.py`, give us idea of [Python Library Hijacking](https://rastating.github.io/privilege-escalation-via-python-library-hijacking/) which I could change path where python will look for by abusing `shutil`.
- When I modify the import directory of Python, and created the python script of my own shutil, I can spawn root access. It's my own python script and I'm making the script to do whatever I want.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/21.png)

- I had to build my own `shutil.py` with the "`make archive`" function (as `make archieve` function will be called from `shutil.py`). But, im not using that method.
- My method is the `shutil.py` must have nc reverse shell that listening on any port pointing to `/bin/bash` shell. (Without forgetting to change PYTHONPATH, Python would first look for the shutil in the directory I specified)
- `sudo - E PYTHONPATH=/tmp /opt/scripts/admin_tasks.sh 6`
- Then, i'm just connect to the victim server to gain root shell.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/admirer/22(root).png)

Voillaa!!
Thanks.
