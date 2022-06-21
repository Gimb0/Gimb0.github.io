---
layout: page
title: Proxmox Cluster Environment
permalink: /about/
---

Creating a Proxmox Cluster Environment.

In this post I will cover my experience creating a Proxmox Cluster environment that will be used for my learnings.

## Hardware

Currently in my Homelab I have 2 Dell R710 Servers that make up my homelab.
I also have a raspberry pi 4 that runs pi-hole and manages my local DNS.
These are in a rack in my garage and are connected by a small 10 port switch.

proxmox1 - 192.168.1.5
proxmox2 - 192.168.1.6

My plan is to configure proxmox2 as the master node

## Issues Connecting to Cluster

The main issue I had while trying to join the proxmox cluster from proxmox1 was:
```console
* local node address: cannot use IP '192.168.1.6', not found on local node!
```

To troubleshoot this I started by testing the network connection.

From proxmox1 I ran:
```console
ping -c 3 192.168.1.6
```
which showed the connection working.

I then tested ssh access by:
```console
root@proxmox1:~# ssh root@192.168.1.6
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:3BjaNMSYV2F6Sk8Att5JL9W8lOgbbIA8HRBcyLdP1t8.
Please contact your system administrator.
Add correct host key in /root/.ssh/known_hosts to get rid of this message.
Offending RSA key in /etc/ssh/ssh_known_hosts:3
  remove with:
  ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "192.168.1.6"
RSA host key for 192.168.1.6 has changed and you have requested strict checking.
Host key verification failed.
```
which showed a certificate issue. This was resolved using the provided command:
```console
ssh-keygen -f "/etc/ssh/ssh_known_hosts" -R "192.168.1.6"
```

Now SSH was working from proxmox1 to proxmox2 and vice versa.
However this did not resolve the error I was receiving while trying to join the cluster.
Looking back at the error "'192.168.1.6', not found on local node!" I started to think it was to do with a system network issue.
I edited the `/etc/hosts` file and added both proxmox servers.

```console
127.0.0.1 localhost.localdomain localhost
192.168.1.5 proxmox1.gimboscloud.dev proxmox1
192.168.1.6 proxmox2.gimboscloud.dev proxmox2

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
```

This change is what fixed my issue with joining the proxmox cluster.