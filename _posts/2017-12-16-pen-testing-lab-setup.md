---
title: Setting up a Penetration Testing Lab
date: 2017-12-16
---

Having just finished up with SANS 560: Network Penetration, I thought I would take the advice of Ed Skoudis (@edskoudis) and set up a small testing lab on my laptop. Given that I was to be jumping onto a plane, I decided that grabbing, the Metasploitable 2 (Linux) image from offensive-security.com as well as the Kali Linux 2017.1 (kali.org) distributions would be my best bet.

After quite a while downloading (slow hotel networks...) I was up and running and ready for my flight!

[...edit...]

Well it turns out that life can really take away all your good intentions. So instead of getting particularly far with any of this setup in July 2017, I didn't end up finishing this write-up until December 2017, when I was getting ready to start the Holiday Hack Challenge.

[...end edit...]

## Kali Linux Investigation

In the SEC 560 class we leveraged the SANS Slingshot 4.1.1 VM, rather than Kali, so this (other than very minimal use in SEC 401 in December 2016) was my first introduction to the operating system.

On the one hand, I was a bit concerned about it (e.g. what is really in this thing?!), and so I started poking at it a bit. Specifically, I was interested in what could get in and out of Kali.

### Updating the defaults

First things first was to update some of the default configurations that are a bit more "security significant". Namely to change the password from the default provided (`toor`), to something that wouldn't be as easily guessed if something was trying to get in:

```
root@kali:~# passwd
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
```

And to update the default host SSH keys being used by the system (https://forums.kali.org/showthread.php?5723-Change-your-Kali-default-ssh-keys):

```
cd /etc/ssh/

# Store the old
mkdir default_kali_keys
mv ssh_host_* default_kali_keys/

# Create the new
dpkg-reconfigure openssh-server
```

### Ingress

Next up, I started looking at what could get in to the host. Given that I intend to use this as a penetration testing operating system, I wanted to know what services were on.

A quick nmap scan of all ports indicated no open ports:

```
root@kali:~# nmap localhost -p0-

Starting Nmap 7.60 ( https://nmap.org ) at 2017-12-16 18:09 EST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0000020s latency).
Other addresses for localhost (not scanned): ::1
All 65536 scanned ports on localhost (127.0.0.1) are closed

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds
```

However, wanting to go a step further, I did a netstat to look for any listening ports, and found nothing but unix file streams:

```
root@kali:~# netstat -an | grep LISTENING
unix  2      [ ACC ]     STREAM     LISTENING     18690    /run/user/0/keyring/pkcs11
unix  2      [ ACC ]     STREAM     LISTENING     19971    @/tmp/.ICE-unix/1002
unix  2      [ ACC ]     STREAM     LISTENING     12096    @irqbalance445.sock
unix  2      [ ACC ]     STREAM     LISTENING     16531    @/tmp/.X11-unix/X0
unix  2      [ ACC ]     STREAM     LISTENING     19862    @/tmp/.X11-unix/X1
unix  2      [ ACC ]     STREAM     LISTENING     14887    @/tmp/dbus-H2dk1GU2jv
unix  2      [ ACC ]     STREAM     LISTENING     14149    @/tmp/.ICE-unix/541
unix  2      [ ACC ]     STREAM     LISTENING     13812    @/tmp/dbus-dfVHhhTl
unix  2      [ ACC ]     STREAM     LISTENING     18633    @/tmp/dbus-38jczD288G
unix  2      [ ACC ]     STREAM     LISTENING     19841    /run/user/0/keyring/control
unix  2      [ ACC ]     STREAM     LISTENING     13809    @/tmp/dbus-PILr4L9I
unix  2      [ ACC ]     STREAM     LISTENING     15504    /run/user/131/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     15509    /run/user/131/gnupg/S.gpg-agent.ssh
unix  2      [ ACC ]     STREAM     LISTENING     15512    /run/user/131/gnupg/S.gpg-agent.extra
unix  2      [ ACC ]     STREAM     LISTENING     15514    /run/user/131/gnupg/S.gpg-agent
unix  2      [ ACC ]     STREAM     LISTENING     19863    /tmp/.X11-unix/X1
unix  2      [ ACC ]     STREAM     LISTENING     15516    /run/user/131/bus
unix  2      [ ACC ]     STREAM     LISTENING     16532    /tmp/.X11-unix/X0
unix  2      [ ACC ]     STREAM     LISTENING     14150    /tmp/.ICE-unix/541
unix  2      [ ACC ]     STREAM     LISTENING     15518    /run/user/131/gnupg/S.dirmngr
unix  2      [ ACC ]     STREAM     LISTENING     18551    @/tmp/dbus-clsV19Bd
unix  2      [ ACC ]     STREAM     LISTENING     15520    /run/user/131/gnupg/S.gpg-agent.browser
unix  2      [ ACC ]     STREAM     LISTENING     19891    /tmp/ssh-h47a7HfKyCgq/agent.1002
unix  2      [ ACC ]     STREAM     LISTENING     12450    /run/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     15522    /run/user/131/pulse/native
unix  2      [ ACC ]     STREAM     LISTENING     18552    @/tmp/dbus-GOGuQCBl
unix  2      [ ACC ]     STREAM     LISTENING     19972    /tmp/.ICE-unix/1002
unix  2      [ ACC ]     STREAM     LISTENING     20646    /run/user/0/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     20651    /run/user/0/gnupg/S.gpg-agent.extra
unix  2      [ ACC ]     STREAM     LISTENING     20654    /run/user/0/gnupg/S.gpg-agent
unix  2      [ ACC ]     STREAM     LISTENING     20656    /run/user/0/gnupg/S.dirmngr
unix  2      [ ACC ]     STREAM     LISTENING     12464    /run/systemd/journal/stdout
unix  2      [ ACC ]     STREAM     LISTENING     20658    /run/user/0/gnupg/S.gpg-agent.ssh
unix  2      [ ACC ]     STREAM     LISTENING     20660    /run/user/0/bus
unix  2      [ ACC ]     STREAM     LISTENING     20662    /run/user/0/pulse/native
unix  2      [ ACC ]     STREAM     LISTENING     20665    /run/user/0/gnupg/S.gpg-agent.browser
unix  2      [ ACC ]     SEQPACKET  LISTENING     12474    /run/udev/control
unix  2      [ ACC ]     STREAM     LISTENING     25264    @/tmp/dbus-ag5xU0qG
unix  2      [ ACC ]     STREAM     LISTENING     13813    @/tmp/dbus-m5rSVIs6
unix  2      [ ACC ]     STREAM     LISTENING     12481    /run/lvm/lvmpolld.socket
unix  2      [ ACC ]     STREAM     LISTENING     12485    /run/systemd/fsck.progress
unix  2      [ ACC ]     STREAM     LISTENING     12498    /run/lvm/lvmetad.socket
unix  2      [ ACC ]     STREAM     LISTENING     25265    @/tmp/dbus-64kDGVTG
unix  2      [ ACC ]     STREAM     LISTENING     13790    /run/NetworkManager/private-dhcp
unix  2      [ ACC ]     STREAM     LISTENING     9969     /run/uuidd/request
unix  2      [ ACC ]     STREAM     LISTENING     9972     /var/run/dbus/system_bus_socket
unix  2      [ ACC ]     STREAM     LISTENING     9975     /var/run/pcscd/pcscd.comm
unix  2      [ ACC ]     STREAM     LISTENING     13810    @/tmp/dbus-Og6MPIHR
unix  2      [ ACC ]     STREAM     LISTENING     18685    /run/user/0/keyring/ssh
root@kali:~#
```

## Setting Up SSH

As part of my setup, I decided that I wanted to enable ssh access from my host into my Kali guest VM. To accomplish this, I needed to start the SSH server, and log in:

```
root@kali:~# service ssh start

# And to confirm that it was running:
root@kali:~# netstat -tlpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1857/sshd
tcp6       0      0 :::22                   :::*                    LISTEN      1857/sshd
```

Great! Now I can go ahead and log in:

```
# From host OS
> ssh root@192.168.16.153
root@192.168.16.153's password:
Permission denied, please try again.
```

Huh... that's weird... I know I'm typing the right password...

Ah, must be something in the `/etc/ssh/sshd_config`! Some poking around in there, found a *very* minimal configuration, which meant that it was the right path, but I must be picking a default `sshd_config` option.. A quick search led me to:

```
root@kali:~# man sshd_config

[...snip...]

     PermitRootLogin
             Specifies whether root can log in using ssh(1).  The argument must be yes, prohibit-password, without-password, forced-commands-only, or no.
             The default is prohibit-password.

             If this option is set to prohibit-password or without-password, password and keyboard-interactive authentication are disabled for root.

[...snip...]
```

So in this case, I'll need to add my host public ssh key, into my Kali Linux VM configuration:

```
# On Host (macOS)
> cat ~/.ssh/id_rsa.pub | pbcopy
```

And then paste that into my running Kali VM, in the appropriate file and directory (which need to be created):

```
root@kali:~# mkdir ~/.ssh
root@kali:~# vim ~/.ssh/authorized_keys
[...paste the copied ssh public key from host...]
```

## Moving On

Well, at this point I've decided that I've down enough typing and setting up, and I'm going to go ahead and jump right in to the 2017 SANS Holiday Hack Challenge, and try my luck against the giant snowballs invading the north pole!

https://www.holidayhackchallenge.com


## Edit (2017-12-29): Enabling the Database

I noticed in pretty short order that the database in Kali isn't running by default and needs to be enabled manually. I found the instructions for doing so at: https://docs.kali.org/general-use/starting-metasploit-framework-in-kali

Which amount to:

```
root@kali:~# systemctl start postgresql
root@kali:~# msfdb init
```
