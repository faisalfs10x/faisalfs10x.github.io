---
layout: single
title: Monteverde write-up
excerpt: "Monteverde was an Active Directory box that requires enumerating user accounts via smb then bruteforce smb login via msf module to log in as user shell. Then we find more credentials by enumerating the machine and abusing Azure Admin to retrieve plain text credential in order to gain Admin shell."
date: 2020-06-15
classes: wide
header:
  teaser: /asset/htbwriteup/windows/monteverde/intro.PNG
  teaser_home_page: true
  icon: /assets/hackthebox.webp
categories:
  - hackthebox
permalink: /htb/htbMonteverde
tags:
  - smb login bruteforce
  - azure ad
  - plaintext creds
  - evil-winrm
  
---

# HTB - Monteverde

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/monteverde/intro.PNG)

Monteverde was an Active Directory box that requires enumerating user accounts via smb then bruteforce smb login via msf module to log in as user shell. Then we find more credentials by enumerating the machine and abusing Azure Admin to retrieve plain text credential in order to gain Admin shell.

---
## Recon

    Running nmap script vulners on selected ports to enumerate the box.
    
![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/monteverde/1.png)

- We started with SMB recon. Let's recon with enum4linux as usual. 

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/monteverde/2.png)

- We found lots of potential usernames for bruteforce. Gathered them all and make username list.

---
## Exploit

- Let's try to bruteforce with `smb_login` module in metasploit. After trying several methods, we came out with `USER_AS_PASS` option and catch up with **_SABatchJobs:SABatchJobs_** as a valid credential.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/monteverde/3.png)

- With the credential we could further our digging into smb shares and connect to share named **_users$_** and we are able to fetch `azure.xml` that lead to another password discovery of `mhope` user.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/monteverde/4.png)

- We have mhope credential now. Just power up `evil-winrm` and login with the credential and catch the user flag.

![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/monteverde/5.0%28user%29.png)

---
## Privilege escalation

- First, we enum the groups information and found that user mhope has group permissions of `MEGABANK\Azure Admins`. There are 2 ways to get admin shell.
- An Azure Administrator is responsible for implementing, monitoring and maintaining Microsoft Azure solutions, including major services related to Compute, Storage, Network and Security.

### 1st way:

- After googling for Azure Admin Privilege escalation we found this [Azure AD Connect Database Exploit](https://github.com/VbScrub/AdSyncDecrypt/releases) that which we could extract plain text password by abusing Azure Admin group.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/monteverde/5.1.png)

- The key role of Azure AD Connect is to synchronize stuff between the local AD domain and Azure domain. However, for that to be feasible it takes privileged access for your local domain to execute certain operations like password syncing, etc.
- This program must be run while the AD Sync Bin folder is your “working directory”, or has been added to the PATH variable. An easy way to do this is simply navigate to the folder in Powershell or Command Prompt (i.e _cd “`C:\Program Files\Microsoft Azure AD Sync\Bin`”_), and then run the program by typing the full path to wherever you have stored it. You also need to make sure the `mcrypt.dll` from the download link is in the same directory the program is in. Failure to do either of these things will result in a Module Not Found error. [-  [vbscrub](https://vbscrub.com/author/vbscrub/ "Posts by vbscrub ( @vbscrub )")  12:57 am  _on_  January 14, 2020 ]
- With powershell, just simple run `AdDecrypt.exe -FullSQL` to get plain text password. Login and you are in admin privilege shell.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/monteverde/6.png)

### 2nd way:

- Based on article of [Azure AD Connect for Red Teamers](https://blog.xpnsec.com/azuread-connect-for-redteam/), to decrypt the `encrypted_configuration` value, the author created a quick POC which will retrieve the keying material from the LocalDB instance before passing it to the `mcrypt.dll` assembly to decrypt and when executed, the decrypted password for the MSOL account will be revealed:
- And what are the conditions for achieving this exfiltration? We'll have to access the LocalDB (if this DB is configured), which includes the following default protection parameters.
- This ensures you have the opportunity to recover the passwords for a DCSync account if you can access the server which contains an Azure AD Connect service to either the ADSyncAdmin or the local Admin community.
- All you have to do is modify the powershell script of `SqlConnection string` to `“Server=LocalHost;Database=ADSync;Trusted_Connection=True;”` that will attempt to access the ADSync database on a full fat MS SQL instance using windows authentication.

![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/monteverde/7.0.png)

- Then upload and run the powershell script to Azure AD server and retrieve the plain text password of Azure Admin.

![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/monteverde/7.2.png)

- Login and get the root flag.

![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/monteverde/7.3%28root%29.jpg)

Thanks.
