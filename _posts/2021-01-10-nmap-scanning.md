---
title: "Nmap - Quick Scanning Tips"
description: "This article covers some basic network scan commands that can be used with nmap, and possibly reduce your time significantly when scanning…"
date: "2021-01-10T15:07:37.922Z"
categories: ["Tech"]
tags: ["Tools"]
published: true
---

## Introduction
This article covers some basic network scan commands that can be used with nmap, and possibly reduce your time significantly when scanning networks.

### What is nmap?

Nmap is a scanning and enumeration tool, that is one of the must-have tools when it comes to penetration testing and red teaming.

### Why should I care?
**This tool is really useful for those who aren’t in cybersecurity as well, as this tool can basically scan for active machines on your local network (home network) and find out if there are any malicious actors (someone who may have compromised your network, say for eg. your neighbor).
Nmap is an extremely powerful tool for security testing and network administration.

 ## Common nmap scans
 An extremely basic nmap scan would be the following. 
```
 > nmap 192.168.119.0/24

 Starting Nmap 7.91 ( [https://nmap.org](https://nmap.org) ) at 2021–01–10 09:50 EST
 Nmap scan report for 192.168.119.2
 Host is up (0.00093s latency).
 Not shown: 999 closed ports
 PORT STATE SERVICE
 53/tcp filtered domain

 Nmap scan report for 192.168.119.128
 Host is up (0.0093s latency).
 Not shown: 999 closed ports
 PORT STATE SERVICE
 8084/tcp open websnp

 Nmap scan report for 192.168.119.129
 Host is up (0.0079s latency).
 Not shown: 994 closed ports
 PORT STATE SERVICE
 22/tcp open ssh
 80/tcp open http
 111/tcp open rpcbind
 139/tcp open netbios-ssn
 443/tcp open https
 32768/tcp open filenet-tms

 Nmap done: 256 IP addresses (3 hosts up) scanned in 4.82 seconds
```
**What did the above scan just do?**

> This scan basically pings your entire local/home network which would comprise IP addresses 192.168.1.0–192.168.1.255 (Your IP Address may vary based on your local network subnet), and checks if they are active.

> If nmap finds out that there are active machines on the network, it would proceed to send probes to find out open/closed ports on those active machines.

The caveat to the above scan is: It takes a long time to complete.

What if you want to ping the entire network, just to find out the active machines in the network? Simple as adding a switch to the nmap command.
```
 > nmap -sn 192.168.119.0/24

 Starting Nmap 7.91 ( [https://nmap.org](https://nmap.org) ) at 2021–01-10 09:47 EST
 Nmap scan report for 192.168.119.2
 Host is up (0.0079s latency).
 Nmap scan report for 192.168.119.128
 Host is up (0.00067s latency).
 Nmap scan report for 192.168.119.129
 Host is up (0.010s latency).
 Nmap done: 256 IP addresses (3 hosts up) scanned in 3.21 seconds
```

> The **-sn** switch basically tells nmap to only do a ping(or host discovery) on all the nodes in a network (192.168.119.0–255). As you can see, the scan completed in a faster time (3.21 seconds).

What if you wanted to run only a port scan, and forget about pinging the hosts on a network? This scenario is applicable, when you know that those hosts are alive, and you don’t want nmap to check if they are alive.

For example, from the above output, the host 192.168.119.128 and 192.168.119.129 machines are alive. We will take 192.168.119.129 as an example here.
```
 > nmap -Pn 192.168.119.129

 Host discovery disabled (-Pn). All addresses will be marked ‘up’ and scan times will be slower.
 Starting Nmap 7.91 ( [https://nmap.org](https://nmap.org) ) at 2021–01–10 10:00 EST
 Nmap scan report for 192.168.119.129
 Host is up (0.0011s latency).
 Not shown: 994 closed ports
 PORT STATE SERVICE
 22/tcp open ssh
 80/tcp open http
 111/tcp open rpcbind
 139/tcp open netbios-ssn
 443/tcp open https
 32768/tcp open filenet-tms

 Nmap done: 1 IP address (1 host up) scanned in 0.27 seconds
```

The above command completely disabled host discovery (shown in output).

The above command printed some output onto the linux terminal. This output could be lost, if you close the terminal!

Of course, there are ways to store the output into a file or files, in the format you specify.

```
 nmap -Pn 192.168.119.129 -oA nmapOutput
 Host discovery disabled (-Pn). All addresses will be marked ‘up’ and scan times will be slower.
 Starting Nmap 7.91 ( [https://nmap.org](https://nmap.org) ) at 2021–01–10 10:04 EST
 Nmap scan report for 192.168.119.129
 Host is up (0.0042s latency).
 Not shown: 994 closed ports
 PORT STATE SERVICE
 22/tcp open ssh
 80/tcp open http
 111/tcp open rpcbind
 139/tcp open netbios-ssn
 443/tcp open https
 32768/tcp open filenet-tms

 Nmap done: 1 IP address (1 host up) scanned in 0.41 seconds
```

The above output should be similar to the previous command, but if you check what files were created (using ls), you would see that 3 files would be created:

```
> ls 
nmapOutput.gnmap nmapOutput.nmap nmapOutput.xml
```
There are 3 files created here.

1.  `gnmap` is short for greppable nmap, which means this could be used in automation scripts to parse the output in a reliable manner.
2.  The second one, is the same as the output shown above.
3.  Finally, XML output could be used for other purposes as well.

This list is very far from being an exhaustive one. If you want to know more about nmap, and its switches, open up a terminal in Linux, and type:

```
man nmap
```

This would display the manual, with all (and more) information that you would need to know about nmap’s functionality.

I hope this article was of some use to you. Feel free to drop comments below.
