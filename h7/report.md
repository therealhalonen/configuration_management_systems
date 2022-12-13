# The Project !
This is the report of my Saltstack project in the course:   
[Configuration Management Systems - Palvelinten Hallinta by Tero Karvinen](https://terokarvinen.com/2022/palvelinten-hallinta-2022p2/)


`I used Debian 11, Lenovo Thinkpad E15`
``` 
~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Debian
Description:	Debian GNU/Linux 11 (bullseye)
Release:	11
Codename:	bullseye
```

### Main sources used, to make this happen:   
[Tero Karvinen](https://terokarvinen.com):
- [Himself in the course lecture](https://terokarvinen.com/2022/palvelinten-hallinta-2022p2/)
- [terokarvinen.com - Everything that includes Salt](https://terokarvinen.com/search/?q=salt)  

[Niko Heiskanen](https://heiskanen.rocks)
- Himself 
- [Reports from previous implementation of the same course.](https://heiskanen.rocks/server_management)

[Miljonka: Mozilla-repo workaround](https://github.com/miljonka/miniproject/blob/main/Starterpack/init.sls)

Class colleagues from the course.   

[Saltproject Documentation](https://docs.saltproject.io/en/latest/index.html)

## Intro
I started to make a project, which includes multiple machines, with different operating systems, and controlling them all with Salt from a single top.sls file.

This project and some of the configurations are meant to be tested in Vagrant environment, for which i also made a Vagrant file.    
Configurations are easily edited for a "real"-, or scaled for bigger environment.    
For example SSH config or the conditional statements for certain operating systems and Minion names.

**Testing of the project can be found [HERE](https://github.com/therealhalonen/configuration_management_systems/blob/master/h7/project_testing.md)**

**Project itself, can be found [HERE](https://github.com/therealhalonen/domain_control)**

## The Project

### States and environment
---
[SaltProject - States](https://docs.saltproject.io/en/latest/ref/states/index.html)   
[My: States](https://github.com/therealhalonen/domain_control/tree/master/srv/salt)   

The default environment includes:   
Production:      
- 2 x Debian 11 servers:
	- fileserver
	- webserver
- Ubuntu 22.04 Desktop:
	- ubuntu-ws
- Windows 10:
	- windows-ws   

Development:
- 1 or more Debian 11 Server(s)
	- dev-server(1-)
- Fedora 36 Desktop
	- dev-fedora-ws
- Ubuntu 22.04 Server
	- dev-ubuntu

States;   
First i made the states i wanted to apply:   
**ssh**
- To get an SSH remote control to servers.
- Change port to 8888
- Allow only user `supreme`

**avahi**
- To servers, in place to mimic DNS.

**apache**
- For Webserver to host a site.

**samba**
- For Fileserver to host files accessible from local machines.
- Accessible as `sambauser`

**pkgs**
- A state which includes software to install.

**update_systems**
- Updates package repositories and installed packages in Linux systems.

**apps**
- State to update winrepo database and install defined software for Windows.    

*These above, are included in top.sls*

These are added as a bonus and debug, and needs to be run if needed, manually:  
**hello_all**
- Legendary 'Hello World', to demonstrate how different operating systems and environments can be handled.

**update_winrepo_ng**
- A bash script ran manually to update winrepo for windows packages
- Only Libreoffice is included from the repo, so running this pulls everything in to the right place to local Salt folder.

**clean_users**
- Needs to be manually run. Not necessary. Clean Vagrant users away, and shouldn't be used if no users are created when Vagrant is used.

Whole local state environment:
`/srv/salt:`
```bash
~$ tree /srv/salt/
/srv/salt/
├── dev
│   └── hello_all
│       └── init.sls
├── prod
│   ├── avahi
│   │   └── init.sls
│   ├── clean_users
│   │   └── init.sls
│   ├── fileserver
│   │   └── samba
│   │       ├── init.sls
│   │       └── smb.conf
│   ├── hello_all
│   │   └── init.sls
│   ├── server_users
│   │   └── init.sls
│   ├── ssh
│   │   ├── init.sls
│   │   └── sshd_config
│   ├── ubuntu
│   │   └── pkgs
│   │       └── init.sls
│   ├── update_systems
│   │   └── init.sls
│   ├── update_winrepo_ng
│   ├── webserver
│   │   └── apache
│   │       ├── init.sls
│   │       └── www
│   │           └── index.html
│   └── win
│       ├── apps
│       │   └── init.sls
│       └── repo-ng
│           ├── remote_map.txt
│           └── salt-winrepo-ng
│               └── libreoffice.sls
└── top.sls
``` 
Okay so as seen in the `tree` there looks to be two environments.   
The states are placed by environment, and by which machine they belong to, or do they belong to every machine. 
Those are explained more detailed in the **Topfile** and **Master Config** -sections

### Master Config
--- 
[SaltProject - Master Config](https://docs.saltproject.io/en/latest/ref/configuration/master.html)   
[My: Master Config](https://github.com/therealhalonen/domain_control/blob/master/etc/salt/master)

In master conf i configured few things to my liking:
- Defined few different Salt Environments = `saltenv`
	-  So that there would be separated states for separated machines to production and development
- One top.sls to control despite the environment
- One environment for Pillar
- Exclude legacy win-repo and define location of win-repo-ng
- Just for testing purposes, defined groups for machines with different conditions.

### Top.sls
---- 
[SaltProject - Topfile](https://docs.saltproject.io/en/latest/ref/states/top.html)   
[My: top.sls](https://github.com/therealhalonen/domain_control/blob/master/srv/salt/top.sls)

To my top.sls, i defined the machines and environments they belong to.   
Also, of course which states are run for which minions with what condition.   
Everything is detailed inside the file with comments, to help remember and understand the infrastructure.

### Pillar
---- 
[SaltProject - Pillar](https://docs.saltproject.io/en/latest/topics/tutorials/pillar.html)   
[My: Pillar](https://github.com/therealhalonen/domain_control/tree/master/srv/pillar)

`/srv/pillar`
```bash
~$ tree /srv/pillar/
/srv/pillar/
├── secret_stuff.sls
└── top.sls
```
Pillars i made, as a learning experience of how they are handled.   
Few states in this project, for example SSH, defines a password, that is included as a hash in `/srv/pillar/secret_stuff.sls` and is applied via Pillars `top.sls` automatically.

### Minion Config
----
*Minion config as it is, is not included.*

In this project, the minion config must include the following, to work as should:
 - IP address of the Master
 - Environment it belongs to
 - Minion id. As seen in Top.sls, its good to be named according to what states it will get.

In this project, they are all put together automatically in Vagrantfile, but if more machines are included, or using this in a real-life environment, they should be configured with those values in mind manually.

For example a new server in Production:
```
#/srv/salt/minion
master: <master address>
saltenv: prod
id: anotherserver
```

### Vagrant - Bonus stuff!
---- 
[Vagrant by HashiCorp](https://www.vagrantup.com/)   
[HashiCorp - Vagrantfile](https://developer.hashicorp.com/vagrant/docs/vagrantfile)   

[My: Vagrantfile, Production](https://github.com/therealhalonen/domain_control/blob/master/vagrant_prod/Vagrantfile)   
[My: Vagrantfile, Development](https://github.com/therealhalonen/domain_control/blob/master/vagrant_dev/Vagrantfile)

This was kind of an optional bonus part, or even a separate project for me.   
Im a huge fan of automation, so i wanted to create a testing ground for my Salt project, which i could build and destroy anytime i wanted.

So after alot of trials and errors, help from Tero Karvinen, my course colleagues and my friend Niko Heiskanen, i managed to put together an all-in-one file to build everything i needed! 

**Its actually split in two different files** and they are included in the project repository.
I wont go through all the process here, but i will explain what it does:   
With the mentioned files, founded behind the link, i/you can just do:   
```vagrant up```
And the other one will create:    
Production   
- 2x Debian 11 - servers
- 1x Ubuntu 22.04 Desktop
- 1x Official free from Microsoft `EdgeOnWindows10` 

While the other creates:   
Development   
- 2x Debian 11 - servers
- 1x Fedora 36 -server
- 1x Ubuntu 22.04 - server   

For Virtualbox.   
It will also define the master, and configure each machine as a minion, and defines the minion config as it should be for the project to work.

Reason i split them in two files, is for the live demonstration.   
They need so much resources from my laptop, so i run the Development VM:s in different PC and in different network, still of course with a single Master to rule them all.   
And also in live demo, i /will use/used/ Fedora Desktop 36 instead of Server, which i build for Vagrant from Virtualbox installed image, but that shouldnt make any difference, as that was also just a learning experience for me.


### TODO
----
To further develop this project:
- ufw
- "Something real" for apache2, in webserver. Currently holds 'Hello World' page
- Better user-control
- Sambashare or SFTP, different folders and permissions for different users
- Testing; merging from Development to Production
- Something something mooooooore!
