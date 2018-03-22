---
title: Working through Metasploitable 2
date: 2018-03-xx
---

## Introduction

Metasploitable 2 is an intentionally vulnerable Linux distribution, provided by the folks at Offensive Security, as a training tool for those looking to learn and develop there skills with the Metasploit framework.

-- #TODO: add a link to download.

This is an older environment, based on Ubuntu 8.04. It comes with a default username and password of `msfadmin` / `msfadmin` which can be used for anything really, but which I only used to log in and query the network address of the system (`172.16.243.143`), which was running as a VMWare VM with a host only network connection. This allowed me to then start probing the system from a fresh Kali Linux 2018.1 VM I had installed alongside of the Metasploitable host.

In this particular case, I chose to save time by just logging in to the target VM and enumerating it's IP address, since it should be one of only three hosts on my small host-only network created by VMWare (the others being the host itself, as well as the attacking Kali Linux VM).

Once I had the IP address, I took the step of adding that to my Kali `/etc/hosts` file, so that I could reference it by the name `target` rather than having to type out the IP address for every command.

    echo "172.16.243.143 target" >> /etc/hosts

## Scanning
