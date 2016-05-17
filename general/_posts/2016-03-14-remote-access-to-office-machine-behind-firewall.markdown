---
layout: post
title: "How-to: Remote Access To Office Machine Behind Firewall"
tags: autossh firewall 
---

You want to ssh your office machine/laptop/workstation.

But the machine is behind the company's firewall and you have no admin rights to the router.

You have come to the right page! :)

<br>

### How this works:
SSH traffic will be directed from your EC2 instance (or any public accessible machine) to your office machine via a reverse tunnel.

ssh => [ec2 instance] =====_reverse ssh tunnel_======  [office machine]

or 

ssh => [publicly accessible machine]  =====_reverse ssh tunnel_======  [office machine]


### You will need:
* a stable office internet connection
* Amazon ec2 instance or publicly accessible machine (with [OpenSSH server][1] installed)
* OpenSSH server installed on office machine 
* [autossh][2]




### Run the following on your office machine:
```bash
# this creates a reverse tunnel between your office machine and your EC2 instance
# ServerAliveInterval 60 means autossh will check for reverse tunnel status every 60 seconds
# ServerAliveCountMax 1000000 means autossh will retry 1million times to create reverse tunnel

autossh -M 0 -R 2222:localhost:22 -i $PATH_TO_YOUR_KEYFILE ubuntu@$EC2_INSTANCE_PUBLIC_IP -o "ServerAliveInterval 60" -o "ServerAliveCountMax 1000000"
```

### Run the following on your EC2 instance
```bash
# this routes all traffic going into your EC2 instance port 2222 to the reverse tunnel created above
# ServerAliveInterval 60 means autossh will check for reverse tunnel status every 60 seconds
# ServerAliveCountMax 1000000 means autossh will retry 1million times to create reverse tunnel

sudo autossh -M 0 -L $EC2_INSTANCE_PRIVATE_IP:2222:localhost:2222 -N 127.0.0.1 -i $PATH_TO_YOUR_KEYFILE -l ubuntu -o "ServerAliveInterval 60" -o "ServerAliveCountMax 1000000"
```

### Enjoy access to your office machine:
``` bash
ssh $OFFICE_MACHINE_USERNAME@$EC2_INSTANCE_PUBLIC_IP -p 2222
```


[1]: https://help.ubuntu.com/community/SSH/OpenSSH/Configuring
[2]: http://www.harding.motd.ca/autossh/
[3]: http://supervisord.org/



