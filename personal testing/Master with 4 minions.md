# Master with 4 minions

*Did this, when i tested, if i could clone my existing Ubuntu Server virtual machine to be used as a several minions. This is not a guide, please dont "read and do", there are tips in the end!*

```
Master: Debian 11 (Virtualbox)
Minion(s): 4x Ubuntu Server 22.10 (Virtualbox)
```
I had a one Ubuntu Server 22.10 installed and ready configured as a minion, in Virtualbox for testing purposes. It already had `salt-minion` installed and `/etc/salt/minion` configured to `master: <Master_IP>`.

I made 3 clones of the same machine.
I also made a Nat Network in Virtualbox:

![[res/Pasted image 20221105114853.png]]

Then i connected all 5 (Master = Debian and 4x Minions = Ubuntu Servers) machines to my newly created NAT Network:

![[res/Pasted image 20221105114956.png]]
Problems started right away, when every machine got the same IP...

First i tried randomizing the mac address from every machines settings in Virtualbox -> Network Adapter. I also did this when i cloned the machines, as there is a dropdown meny to do so automatically... But that didnt work.

After that i tried (on my cloned server machine):
```
~$ sudo dhclient -r
~$ sudo dhclient enp0s3
```

To release and obtain new IP from DHCP (`enp0s3` is the network adapter in my virtual machine(s))

That didnt work, so i figured, the dhcp will probably use something else to identify the machine.

Found this article: [Ask Ubuntu](https://askubuntu.com/questions/1179897/ubuntu-18-04-guests-which-cloned-by-virtualbox-have-the-same-ip-but-different-ma)

After trial and error, this worked:
```bash
~$ sudoedit /etc/netplan/*
```
`dhcp-identifier: mac` so dhcp will be forced to use mac-address as an identifier.

![[res/Pasted image 20221105110315.png]]

After that i ran `sudo netplan apply` and voila, new ip was received.
I did this to each 4 servers and also to my master.
To verify they were working, i ran `sudo ifconfig -a` in every machine, and saw they all got a new IP from DHCP, and everyone got a different one.

Then proceeded to config 
In my Master i had already salt-master installed and to all 4 servers had salt-minion as previously explained.

Every server i configured the same:
```bash
~$ sudoedit /etc/salt/minion
```
added: `master: 10.0.2.4`  <- Masters = My Debian machines IP
```bash
~$ sudo systemctl restart salt-minion
```

Okay so... Now i had more issues... Every server had the same hostname, due to cloning. So i changed them! 

Done, again, to every server (1-4):
```bash
~$ sudo hostnamectl set-hostname ubnt_serv<1-4>
```
After prompting authentication, they all had a new hostname, and i rebooted them.
Next up, was the issue, that all of them got their minion ID from the hostname, which was configured before i changed the hostnames.
On my master i noticed that:
```bash
~$ sudo salt-key
Accepted Keys:
Denied Keys:
Unaccepted Keys:
exp
Rejected Keys: 
```

`exp` was the hostname of the `Server1` from which i made the 3 clones.
So i had to give them each new Minion IDs.
This was done with ( again to every server separately ):

[Source and pointers](https://stackoverflow.com/questions/47648183/how-to-seamlessly-rename-a-minion)
```bash
sudoedit /etc/salt/minion_id
```
And replacing my "exp" with `Server<1-4>`

After that, of course again i had to kick the minion daemon.
```bash
sudo systemctl restart salt-minion
```
I think i had to also kick my master... Im not sure but anyway on my Master:
```bash
sudo systemctl restart salt-master
```

And then i checked my pending keys (on my master):
```bash
~$ sudo salt-key
Accepted Keys:
Denied Keys:
Unaccepted Keys:
Server1
Server2
Server3
Server4
exp
```

Next i deleted the `exp` key and accepted the rest:
```bash
~$ sudo salt-key -d exp
The following keys are going to be deleted:
Unaccepted Keys:
exp
Proceed? [N/y] y
Key for minion exp deleted.
~$ sudo salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
Server1
Server2
Server3
Server4
Proceed? [n/Y] y
Key for minion Server1 accepted.
Key for minion Server2 accepted.
Key for minion Server3 accepted.
Key for minion Server4 accepted.
```

Done and went to the quick test.
I tried pinging them with salt, first only Server1 for testing and learning purposes:
```bash
~$ sudo salt Server1 test.ping
Server1:
    True
```
Working!
And then all of them:
```bash
~$ sudo salt '*'  test.ping
Server2:
    True
Server4:
    True
Server3:
    True
Server1:
    True
```
YESSS!! Profit!

My tips for cloning a Ubuntu Server 22.10 machine, to use as a minion:
- Remove salt-minion before cloning a machine. Install and configure it to each machine separately.
- Clone and configure only 1 machine at a time! You save time and nerves!
- If you want to use Nat Network -option, configure netplan, to `dhcp-indentifier: mac` before cloning. And when cloning, use `Mac Address Policy: Generate new MAC addresses for all network adapters`
- Remember to change `hostname` before installing and configuring `salt-minion` 
- Last tip: Read and learn how to use [Vagrant](https://www.vagrantup.com/) instead :)

