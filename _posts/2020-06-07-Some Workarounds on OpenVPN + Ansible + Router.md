---
layout: post
title: Some Workarounds on OpenVPN + Ansible + Router
date: 2020-06-07 15:23
category: 
author: 
tags: []
summary: 
---

I want to automate OpenVPN setup using Ansible Roles. This post includes some failure experiences, it's not directly about success, just want to log them. My target server is an EC2 instance under an AWS free account.

First I tried [Stouts/Stouts.openvpn](https://github.com/Stouts/Stouts.openvpn), followed its newest example, but failed on the final step, generate clients keys. The error I got is like this issue: [openssl.cnf file could be found · Issue #110 · Stouts/Stouts.openvpn](https://github.com/Stouts/Stouts.openvpn/issues/110).

I searched some resources, but this role depends on another role [nkakouros-original/ansible-role-easyrsa](https://github.com/nkakouros-original/ansible-role-easyrsa) to tackle easyrsa. It's hard to simple modify the Ansible code to solve the issue without manually login to server.

Even I followed the step from here([OpenVPN server on Debian – Ronan's blog](https://www.fontenay-ronan.fr/openvpn-server-on-debian/))
```
ln -s openssl-1.0.0.cnf openssl.cnf
```
I still cannot generate keys even running command manually on the server, like described here [Openvpn -- unable to generate keys](https://openvpn-users.narkive.com/gca7R057/openvpn-unable-to-generate-keys). Also failed trying the steps here [How to add a user key pair to openvpn without remaking ca.crt - Ask Ubuntu](https://askubuntu.com/questions/945592/how-to-add-a-user-key-pair-to-openvpn-without-remaking-ca-crt).

I don't manually want to setup from the beginning. I tried another Ansible Role: [kyl191/ansible-role-openvpn: Ansible Playbook for OpenVPN on CentOS/Fedora/RHEL clones](https://github.com/kyl191/ansible-role-openvpn). It did everything successfully without any dependencies. Before connecting to OpenVPN, need to open the port by changing Security Group in AWS.

But I encountered another issue while I tried to connect to OpenVPN from the router. I used the same profile which was successfully connected on Windows. Even the OpenVPN client version is higher on the router than on Windows. First issue I ran into was due to option `register-dns`. I solved this issue by change to line to `setenv opt register-dns` from this ticket: [#570 (Options error: Unrecognized option or missing parameter(s) in [PUSH-OPTIONS]:4: register-dns (2.3.6)) – OpenVPN Community](https://community.openvpn.net/openvpn/ticket/570).

Then another issue emerged, failed to get response with error message after connected:
```
write to TUN/TAP : Invalid argument (code=22)
```

From some discussions, it's related to compress option:
* [[SOLVED] [ArchLinux] invalid argument code 22 - OpenVPN Support Forum](https://forums.openvpn.net/viewtopic.php?t=9980)
* [#533854 - openvpn: write to TUN/TAP : Invalid argument (code=22) - Debian Bug report logs](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=533854)

I'm not sure if the warning message at the beginning of the logs caused this:
```
asus WARNING: 'comp-lzo' is present in remote config but missing in local config
```
Even I added the option, still got this warning. Maybe I used multiple VPN connection on the router made the problem. But from this post [Warnings - OpenVPN Support Forum](https://forums.openvpn.net/viewtopic.php?t=26061), someone replied: 
> I run multiple VPNs and they all have this MTU warning .. I don't believe it is worth resolving.

For now, I just disabled compression option on both server and client side, then it works. Final script used in Ansible playbook is just this:
```
  roles:
    - {
        role: kyl191.openvpn, 
        clients: [myvpn],
        openvpn_compression: ''
      }
```

Need to install the role by Ansible Galaxy first. Ansible version I'm using is: 
```
ansible 2.9.9
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/o/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/o/.local/lib/python3.6/site-packages/ansible
  executable location = /home/o/.local/bin/ansible
  python version = 3.6.9 (default, Apr 18 2020, 01:56:04) [GCC 8.4.0]
```