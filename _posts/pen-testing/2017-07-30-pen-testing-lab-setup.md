---
title: Setting up a Penetration Testing Lab
date: 2017-08-04
---

Having just finished up with SANS 560: Network Penetration, I thought I would take the advice of Ed Skoudis (@edskoudis) and set up a small testing lab on my laptop. Given that I was to be jumping onto a plane, I decided that grabbing, the Metasploitable 2 (Linux) image from offensive-security.com as well as the Kali Linux 2017.1 (kali.org) distributions would be my best bet.

After quite a while downloading (slow hotel networks...) I was up and running and ready for my flight!

## Kali Linux Investigation

In the SEC 560 class we leveraged the SANS Slingshot 4.1.1 VM, rather than Kali, so this (other than very minimal use in SEC 401 in December 2016) was my first introduction to the operating system.

On the one hand, I was a bit concerned about it (e.g. what is really in this thing?!), and so I started poking at it a bit. Specifically, I was interested in what could get in and out of Kali.

I started looking at what could get in. Given that I intend to use this as a penetration testing operating system, I wanted to know what services were on.

A quick nmap scan of all ports indicated no open ports:

```
root@kali:~# nmap localhost -p0-

Starting Nmap 7.40 ( https://nmap.org ) at 2017-07-30 13:21 EDT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0000020s latency).
Other addresses for localhost (not scanned): ::1
All 65536 scanned ports on localhost (127.0.0.1) are closed

Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds
```

However, wanting to go a step further, I did a netstat to look for any listening ports, and found nothing but unix file streams:

```
root@kali:~# netstat -an | grep LISTENING
unix  2      [ ACC ]     STREAM     LISTENING     11006    /run/uuidd/request
unix  2      [ ACC ]     STREAM     LISTENING     16379    @/tmp/.X11-unix/X0
unix  2      [ ACC ]     STREAM     LISTENING     16425    /run/user/131/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     16430    /run/user/131/gnupg/S.gpg-agent
unix  2      [ ACC ]     STREAM     LISTENING     17827    @/tmp/dbus-wnlM1qh4lj
unix  2      [ ACC ]     STREAM     LISTENING     16433    /run/user/131/gnupg/S.gpg-agent.ssh
unix  2      [ ACC ]     STREAM     LISTENING     16435    /run/user/131/bus
unix  2      [ ACC ]     STREAM     LISTENING     16437    /run/user/131/gnupg/S.gpg-agent.browser
unix  2      [ ACC ]     STREAM     LISTENING     16366    @/tmp/dbus-sg3Bj4fd
unix  2      [ ACC ]     STREAM     LISTENING     16439    /run/user/131/gnupg/S.gpg-agent.extra
unix  2      [ ACC ]     STREAM     LISTENING     14311    @/tmp/.ICE-unix/927
unix  2      [ ACC ]     STREAM     LISTENING     15867    @/tmp/.ICE-unix/644
unix  2      [ ACC ]     STREAM     LISTENING     13653    /run/user/131/wayland-0
unix  2      [ ACC ]     STREAM     LISTENING     14191    /run/user/0/keyring/control
unix  2      [ ACC ]     STREAM     LISTENING     15662    @/tmp/dbus-cEWhDVpt
unix  2      [ ACC ]     STREAM     LISTENING     15868    /tmp/.ICE-unix/644
unix  2      [ ACC ]     STREAM     LISTENING     13643    /tmp/.X11-unix/X1024
unix  2      [ ACC ]     STREAM     LISTENING     15229    /var/run/NetworkManager/private-dhcp
unix  2      [ ACC ]     STREAM     LISTENING     16380    /tmp/.X11-unix/X0
unix  2      [ ACC ]     STREAM     LISTENING     18551    /tmp/ssh-5p5NdSbKGZUv/agent.927
unix  2      [ ACC ]     STREAM     LISTENING     14312    /tmp/.ICE-unix/927
unix  2      [ ACC ]     STREAM     LISTENING     17538    /run/user/131/pulse/native
unix  2      [ ACC ]     STREAM     LISTENING     13671    @/tmp/dbus-FIRssqvlPk
unix  2      [ ACC ]     STREAM     LISTENING     11411    /run/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     11416    /run/lvm/lvmetad.socket
unix  2      [ ACC ]     STREAM     LISTENING     11420    /run/lvm/lvmpolld.socket
unix  2      [ ACC ]     STREAM     LISTENING     15665    @/tmp/dbus-WlWKBFPX
unix  2      [ ACC ]     STREAM     LISTENING     15663    @/tmp/dbus-slctHrkM
unix  2      [ ACC ]     SEQPACKET  LISTENING     11426    /run/udev/control
unix  2      [ ACC ]     STREAM     LISTENING     11432    /run/systemd/journal/stdout
unix  2      [ ACC ]     STREAM     LISTENING     11449    /run/systemd/fsck.progress
unix  2      [ ACC ]     STREAM     LISTENING     16322    /run/user/0/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     16327    /run/user/0/gnupg/S.gpg-agent
unix  2      [ ACC ]     STREAM     LISTENING     16330    /run/user/0/bus
unix  2      [ ACC ]     STREAM     LISTENING     16332    /run/user/0/gnupg/S.gpg-agent.extra
unix  2      [ ACC ]     STREAM     LISTENING     16334    /run/user/0/gnupg/S.gpg-agent.browser
unix  2      [ ACC ]     STREAM     LISTENING     16336    /run/user/0/pulse/native
unix  2      [ ACC ]     STREAM     LISTENING     16367    @/tmp/dbus-kPPu0QX1
unix  2      [ ACC ]     STREAM     LISTENING     13642    @/tmp/.X11-unix/X1024
unix  2      [ ACC ]     STREAM     LISTENING     16339    /run/user/0/gnupg/S.gpg-agent.ssh
unix  2      [ ACC ]     STREAM     LISTENING     15666    @/tmp/dbus-08BLdjQF
unix  2      [ ACC ]     STREAM     LISTENING     14323    /run/user/0/keyring/ssh
unix  2      [ ACC ]     STREAM     LISTENING     14325    /run/user/0/keyring/pkcs11
unix  2      [ ACC ]     STREAM     LISTENING     11000    /var/run/dbus/system_bus_socket
unix  2      [ ACC ]     STREAM     LISTENING     11003    /var/run/pcscd/pcscd.comm
```
