# h6 Kulkurin projekti
*Homeworks of sixth class.   
Creating virtual machines quickly with Vagrant.   
Automating Salt Master/Minion installation, for virtual machines created with Vagrant.   
Starting my own project for the course.

`I used Debian 11, Lenovo Thinkpad E15`
```bash
~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Debian
Description:	Debian GNU/Linux 11 (bullseye)
Release:	11
Codename:	bullseye
```

### x) Lue ja tiivistä (muutamalla ranskalaisella viivalla per artikkeli, poimi esim itsellesi keskeisimmät komennot)

- **Karvinen 2017: [Vagrant Revisited – Install & Boot New Virtual Machine in 31 seconds](https://terokarvinen.com/2017/04/11/vagrant-revisited-install-boot-new-virtual-machine-in-31-seconds/) (Suosittelen käyttämään tässä koneena 'vagrant init debian/bullseye64')**
	-  Easiest way to install a virtual machine, is with Vagrant (Virtualbox needed first).
		- `sudo apt-get update && sudo apt-get -y install vagrant`
		- After installation, in your $PWD: 
			- ```vagrant init debian/bullseye64```
			- ```vagrant up```
			- And you have a Debian 11 Server VM ready!
			- To destroy them: ```vagrant destroy``` 
- **Karvinen 2021: [Two Machine Virtual Network With Debian 11 Bullseye and Vagrant](https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/)**
	-  After initiating the machine, as mentioned in previous article, you can configure all kinds of stuff in the Vagrantfile. All the configs are inside the file, commented, so its pretty easy to figure out, what they do.
	- You can even configure the single Vagrant file to create multiple machines at the same time and configure them to add them to the same network together.
	- You dont need to do any `init` if you now your way around how the Vagrantfile works. You can make it from scratch!

- **Karvinen 2018: [Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux](https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/)**
	- Configuring two Linux machines to the Master-Minion architecture, is as easy as installing `salt-master` to one and `salt-minion` to other(s).
	- Both can be installed usually from default package manager repository, but its recommended to use [SaltProject Repo](https://repo.saltproject.io/) , to get the latest version, as the version should be same in Master and Minion. 
		- Some distros for example Ubuntu and Debian have different version when installing default.
		- Master should always be newer then Minion, to avoid conflicts and errors.
	- To make the connection between Master and Minion, after installing whats needed, only Minion needs to know whos the Master.
		- That is achieved by configurin Minions config at `/etc/salt/minion` and adding `master: <ip-address or hostname>`
		- After that Master can see Minion offering his key with `sudo salt-key` and accepting it with `sudo salt-key -a <minion_id>`
---
### Sources:
- **Articles above**
- **[Tero Karvinen himself, in the course lecture](https://terokarvinen.com/2022/palvelinten-hallinta-2022p2)** 
- **My friend [Niko Heiskanen himself](https://heiskanen.rocks/)**
- **[Vagrant Documentation](https://developer.hashicorp.com/vagrant/docs)**    
**has been used as a source for this homework.**

### a) Hello Vagrant. Asenna virtuaalikone Vagrantilla.

First got the latest version of Vagrant, by adding the sources:   
```bash
 wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
 echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```
Then installed it:
```bash
sudo apt update && sudo apt install vagrant
```

After that i could create a folder, and inside the created folder, for example to create a Debian 11 - Server machine:
```bash
vagrant init debian/bullseye64 # This creates Vagrantfile, which holds all the configurations for the machine.
vagrant up # This builds or starts the machine configured in previous step.
```
But i did this little bit different...

---
### b) Yksityisverkko. Asenna kaksi virtuaalikonetta samaan verkkoon Vagrantilla. Laita toisen koneen nimeksi "isanta" ja toisen "renki1". Kokeile, että "renki1" saa yhteyden koneeseen "isanta" (esim. ping tai nc). Tehtävä tulee siis tehdä alusta, vaikka olisit ehtinyt kokeilla tätä tunnilla.

### c) Salt master-slave. Toteuta Salt master-slave -arkkitehtuuri verkon yli. Aseta edellisen kohdan kone renki1 orjaksi koneelle isanta.

*I took the rights to bundle these two assignments together and automate them.    
This is done, with learning in mind;   
I wanted to learn, how to do this kind of stuff as efficient as i could.*   

**Warning!  
Dont follow my lead! Do everything manually first, and when you understand how `Vagrantfile` works, you can start experimenting stuff!    
Take backups and safety precautions, this is gonna get wild!** 

### Creating two Debian 11 -servers, to same network and configuring Master-Minion architecture between them, with Vagrant... Automatically!!

**Lock and load!**   
So ive previously tested plenty of configurations in Vagrantfiles and now made my own, with the specs defined in the assignements:   
```ruby
# Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Install salt-master to "isanta" machine
# These provision commands/scripts could be also run locally in the machine, but this automates it when it first creates them.
$master = <<MASTER
apt update -y
apt install salt-master -qy
systemctl restart salt-master
MASTER

# Install salt-minion to "renki1" machine, and config it, to make a connection with "isanta"
$minion = <<MINION
# These could be run locally also in the machine, but this automates it when it first creates it.
apt update -y
sudo apt-get install salt-minion -qy
# This adds the Masters IP to minion -config file, so Minion is instantly calling the right Master, which is gonna be created before this is ran.
sed -i 's/\#master\:.*/master\:\ 192.168.56.66/g' /etc/salt/minion
systemctl restart salt-minion
MINION

Vagrant.configure(2) do |config|
  # Define specs for "isanta"
  config.vm.define "isanta" do |master|
  # Define OS
    master.vm.box = "debian/bullseye64"
    # Give it a hostname defined in the assignment
    master.vm.hostname = "isanta"
    # Add a second NIC and config it as a Virtualbox Host-Only, with static IP
    master.vm.network :private_network, ip: "192.168.56.66"
    # Run the script in the machine, that installs the salt-master
    master.vm.provision "shell", inline: $master
    # Disable shared folders in Virtualbox, to speed up the process
	master.vm.synced_folder '.', '/vagrant', disabled: true
	# Define Virtualbox settings=specs for the machine
    master.vm.provider :virtualbox do |vb|
      vb.name = "isanta"
	  vb.memory = 1028
      vb.cpus = 2
	  vb.customize ["modifyvm", :id, "--vram", "16"]
    end
  end
   # Define specs for "renki1"
  config.vm.define "renki1" do |minion|
	# Define OS
    minion.vm.box = "debian/bullseye64"
    # Give it a name hostname defined in the assignment
    minion.vm.hostname = "renki1"
    # Add a second NIC and config it as a Virtualbox Host-Only, with static IP
    minion.vm.network :private_network, ip: "192.168.56.67"
    # Run the script in the machine, that installs the salt-minion and adds Master's IP to minion -config file.
    minion.vm.provision "shell", inline: $minion
    # Disable shared folders in Virtualbox, to speed up the process
	minion.vm.synced_folder '.', '/vagrant', disabled: true
	# Define Virtualbox settings=specs for the machine
    minion.vm.provider :virtualbox do |vb|
      vb.name = "renki1"
	  vb.memory = 1028
      vb.cpus = 2
	  vb.customize ["modifyvm", :id, "--vram", "16"]
    end
  end
end
``` 

To break it down, i commented everything inside the code.   

Testing:   
As i already had the `debian/bullseye64` automatically downloaded, this was a quick process: 
Using a `temp` folder for testings, and code above, inside a Vagrantfile in the folder:   
```bash
~/temp$ vagrant up
Bringing machine 'isanta' up with 'virtualbox' provider...
Bringing machine 'renki1' up with 'virtualbox' provider...
==> isanta: Importing base box 'debian/bullseye64'...
#
# ...Long list of stuff...
#
==> renki1: Importing base box 'debian/bullseye64'...
#
# ...Long list of stuff...
#
==> isanta: Machine 'isanta' has a post `vagrant up` message. This is a message
==> isanta: from the creator of the Vagrantfile, and not from Vagrant itself:
==> isanta: 
==> isanta: Vanilla Debian box. See https://app.vagrantup.com/debian for help and bug reports

==> renki1: Machine 'renki1' has a post `vagrant up` message. This is a message
==> renki1: from the creator of the Vagrantfile, and not from Vagrant itself:
==> renki1: 
==> renki1: Vanilla Debian box. See https://app.vagrantup.com/debian for help and bug reports
```
Done!   

Outcome in Virtualbox:     
![Image1](https://github.com/therealhalonen/configuration_management_systems/blob/master/h6/res/virtualbox.png)

Then, using Vagrants SSH, i checked that everything is as they should:   
Master = `isanta`: 
```
~/temp$ vagrant ssh isanta
Linux isanta 5.10.0-18-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
vagrant@isanta:~$ sudo systemctl status salt-master
● salt-master.service - The Salt Master Server
     Loaded: loaded (/lib/systemd/system/salt-master.service; enabled; vendor p>
     Active: active (running) since Sat 2022-12-03 09:10:27 UTC; 2min 43s ago
       Docs: man:salt-master(1)
             file:///usr/share/doc/salt/html/contents.html
             https://docs.saltstack.com/en/latest/contents.html
   Main PID: 2555 (salt-master)
      Tasks: 32 (limit: 1133)
     Memory: 204.6M
        CPU: 9.384s
```
Connection to Minion= `renki1`:
```bash
vagrant@isanta:~$ ping -c 1 192.168.56.67
PING 192.168.56.67 (192.168.56.67) 56(84) bytes of data.
64 bytes from 192.168.56.67: icmp_seq=1 ttl=64 time=0.714 ms

--- 192.168.56.67 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.714/0.714/0.714/0.000 ms
```
**Working!**  
And obvious test:  Is Master-Minion architecture working already?   
```bash
vagrant@isanta:~$ sudo salt-key
Accepted Keys:
Denied Keys:
Unaccepted Keys:
renki1
Rejected Keys:
```
**MUAAAHAHAHAAA!! ITS ALIVEEEEE!**   

More testings:   
Accepted the key, and ran a few commands:   
```bash
vagrant@isanta:~$ sudo salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
renki1
Proceed? [n/Y] y
Key for minion renki1 accepted.
vagrant@isanta:~$ sudo salt renki1 cmd.run "echo Im here!"
renki1:
    Im here!
vagrant@isanta:~$ sudo salt renki1 grains.item host
renki1:
    ----------
    host:
        renki1
vagrant@isanta:~$ sudo salt renki1 grains.item ipv4|grep 192
		- 192.168.56.67
```
**Uuyeah! Working!**
Theres no need to check ports, or test connections, as it shows everything is established, but due to assignment, i did it anyway!

Minion= `renki1`
```bash
~/temp$ vagrant ssh renki1
Linux renki1 5.10.0-18-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
vagrant@renki1:~$ nc -nvz 192.168.56.66 4505-4506, 22
(UNKNOWN) [192.168.56.66] 4506 (?) open
(UNKNOWN) [192.168.56.66] 4505 (?) open
(UNKNOWN) [192.168.56.66] 22 (ssh) open
vagrant@renki1:~$ nc -nvz 192.168.56.66 21
(UNKNOWN) [192.168.56.66] 21 (ftp) : Connection refused
```
**So ports for Salt and SSH are open in Master, and ftp is gladly closed!**
**Mission accomplished!**

### d) Oma suola. Tee ensimmäinen työversio projektistasi. Miniprojektilla tulee olla jokin tarkoitus, vaikka se olisi keksitty. Projektilla tulee olla sivu (esim. Github, Gitlab...), josta selviää projektin perustiedot. Toiminnallisuutta tulee olla kokeiltu, mutta sen ei tarvitse olla valmis. Valmiit projektit esitellään viimeisellä tapaamiskerralla. Tässä tehtävässä palautettava työversio ei siis ole vielä lopullinen.

**So this is the course Final, my own project with Salt.   
I had an idea, or actually an obsession of controlling several machines, possibly even having different OS:s installed, and configuring them from Linux, with Salt.**

*Due to schedules and timeframe of the project, i decided to use only Debian based distros for the Linux machines, as those im more familiar with and dont have to start from nearly basic stuff with Red Hat family. Windows i took, for the challenge....*

So some specs:
All machines in same network: 
- Debian 11 Servers
	- Fileserver
	- Webserver
- Ubuntu Desktops as Workstations
- Windows 10 as Workstations

Multiple states for each specific operation systems and according to need.

---
I started the project with creating the actual testing environment for myself.   
I used Virtualbox VM:s with [Vagrant](https://www.vagrantup.com/)

My Vagrantfile at this point, while writing this, looks like this:
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# WIP: EXPERIMENTAL SCRIPTS!!! USE WITH CAUTION, OR DONT USE AT ALL :D!

# Sources:
# https://gist.github.com/santrancisco/a7183470efa0e3412222670d0bfb3da5
# Tero Karvinen Himself and; https://terokarvinen.com/2018/automatically-provision-vagrant-virtualmachines-as-salt-slaves/
# Niko Heiskanen: https://heiskanen.rocks/server_management/h1 

# Provision Linux:s
$linux = <<LINUX
apt update -y
sudo apt install bash-completion -y
apt install salt-minion -qy
sed -i 's/\#master\:.*/master\:\ 192.168.56.1/g' /etc/salt/minion
systemctl restart salt-minion
LINUX

# Provision Windows
$windows= <<WINDOWS
echo Installing Salt-Minion.
echo This will take a while, so dont worry!
powershell.exe -Command "[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; 
Invoke-WebRequest -Uri 'https://repo.saltproject.io/windows/Salt-Minion-3004.2-1-Py3-AMD64.msi' -OutFile 'C:/Users/IEUser/Salt-Minion-3004.2-1-Py3-AMD64.msi';
msiexec /i Salt-Minion-3004.2-1-Py3-AMD64.msi /quiet /norestart MASTER=192.168.56.1 MINION_ID=windows-ws"
waitfor WINDOWStoConfigureSalt /t 30 2>NUL
echo Done!
WINDOWS

Vagrant.configure(2) do |config|

  # Webserver
  config.vm.define "webserver" do |ws|
    ws.vm.box = "debian/bullseye64"
    ws.vm.hostname = "webserver"
    ws.vm.network :private_network, ip: "192.168.56.10"
    ws.vm.provision "shell", inline: $linux
	ws.vm.synced_folder '.', '/vagrant', disabled: true
    ws.vm.provider :virtualbox do |vb|
      vb.name = "Webserver"
	  vb.memory = 512
      vb.cpus = 1
	  vb.customize ["modifyvm", :id, "--vram", "8"]
    end
  end    
  # Filerserver
  config.vm.define "fileserver" do |fs|
    fs.vm.box = "debian/bullseye64"
    fs.vm.hostname = "fileserver"
    fs.vm.network :private_network, ip: "192.168.56.11"
    fs.vm.provision "shell", inline: $linux
	fs.vm.synced_folder '.', '/vagrant', disabled: true
    fs.vm.provider :virtualbox do |vb|
      vb.name = "Fileserver"
	  vb.memory = 512
      vb.cpus = 1
	  vb.customize ["modifyvm", :id, "--vram", "8"]
    end
  end
  
  # Ubuntu - Workstation
  config.vm.define "ubuntu-ws" do |ubuntu|
    ubuntu.vm.box = "fasmat/ubuntu2204-desktop"
    ubuntu.vm.box_version = "22.0509.1"
    ubuntu.vm.hostname = "ubuntu-ws"
    ubuntu.vm.network :private_network, ip: "192.168.56.101"
    ubuntu.vm.provision "shell", inline: $linux
	ubuntu.vm.synced_folder '.', '/vagrant', disabled: true
    ubuntu.vm.provider :virtualbox do |vb|
	  vb.linked_clone = false
      vb.name = "Ubuntu-Ws"
	  vb.memory = 1028
      vb.cpus = 2
      vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
	  vb.customize ["modifyvm", :id, "--vram", "32"]
	  vb.gui = false
    end
  end  
  
  # Windows - Workstation
  config.vm.define "windows-ws" do |window|  

  # Basic
	window.vm.box = "Microsoft/EdgeOnWindows10"
	window.vm.guest = :windows 
	window.vm.synced_folder '.', '/vagrant', disabled: true

  # Network
	window.vm.network :private_network, ip: "192.168.56.102", auto_config: false
	
  # Config SSH
	window.ssh.username="IEUser"
	window.ssh.password="Passw0rd!"
	window.ssh.insert_key = false
	window.ssh.sudo_command = ''
	window.ssh.shell = 'sh -l'

  # Machine stuff
    window.vm.provider :virtualbox do |vb|
      vb.name = "windows-wm"
	  vb.memory = 4096
      vb.cpus = 2
      vb.customize ["modifyvm", :id, "--graphicscontroller", "vboxsvga"]
	  vb.customize ["modifyvm", :id, "--vram", "128"]
	  vb.gui = false
	end

  # Install and configure Salt-Minion
  	window.vm.provision "shell", privileged: false, inline: $windows
	end
end
``` 
And can be set up, down, destroy with simply:
```bash
vagrant up 
vagrant halt
vagrant destroy
```
So as for what i  needed for this project, it creates:
- 2x Debian 11 - Servers
- 1x Ubuntu 22.04 -Desktop 
- 1x Official 'MSEdge on Win10 X64' 
Automates Salt-minion installation on everyone of them and defines the address of the Master, which in this case is my Host machine, mentioned at the beginning.


**To the actual Project:**

The local Master's `/srv/salt` where all the statefiles are, looks like this, at this point:
```bash
~$ tree /srv/salt
/srv/salt
├── fileserver
│   └── samba
│       ├── init.sls
│       └── smb.conf
├── hello_all
│   └── init.sls
├── top.sls
├── ubuntu
│   └── pkgs
│       └── init.sls
├── update_systems
│   └── init.sls
├── update_winrepo_ng
├── webserver
│   └── apache
│       ├── init.sls
│       └── www
│           └── index.html
└── win
    ├── apps
    │   └── init.sls
    └── repo-ng
        ├── remote_map.txt
        └── salt-winrepo-ng
            └── < salt-winrepo-ng content >
```
Quick test:   
```bash
~/Projects/project_folder$ sudo salt '*' state.apply hello_all
[sudo] password for sicki: 
ubuntu-ws:
----------
          ID: echo Hello $(hostname)!
    Function: cmd.run
      Result: True
     Comment: Command "echo Hello $(hostname)!" run
     Started: 13:48:20.047325
    Duration: 8.177 ms
     Changes:   
              ----------
              pid:
                  2032
              retcode:
                  0
              stderr:
              stdout:
                  Hello ubuntu-ws!

Summary for ubuntu-ws
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:   8.177 ms
webserver:
----------
          ID: echo Hello $(hostname)!
    Function: cmd.run
      Result: True
     Comment: Command "echo Hello $(hostname)!" run
     Started: 13:48:23.895795
    Duration: 18.463 ms
     Changes:   
              ----------
              pid:
                  982
              retcode:
                  0
              stderr:
              stdout:
                  Hello webserver!

Summary for webserver
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  18.463 ms
fileserver:
----------
          ID: echo Hello $(hostname)!
    Function: cmd.run
      Result: True
     Comment: Command "echo Hello $(hostname)!" run
     Started: 13:48:23.906026
    Duration: 20.547 ms
     Changes:   
              ----------
              pid:
                  943
              retcode:
                  0
              stderr:
              stdout:
                  Hello fileserver!

Summary for fileserver
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  20.547 ms
windows-ws:
----------
          ID: echo Hello %COMPUTERNAME%!
    Function: cmd.run
      Result: True
     Comment: Command "echo Hello %COMPUTERNAME%!" run
     Started: 05:48:25.037925
    Duration: 91.967 ms
     Changes:   
              ----------
              pid:
                  2764
              retcode:
                  0
              stderr:
              stdout:
                  Hello IE11WIN10!

Summary for windows-ws
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  91.967 ms
```
**Hello World working!**   
Thats pretty much everything that should be documented at this point, as so much stuff can change after taking notes, hints and tips from next class @ 8.12.2022.   
The functions are tested and at this point they work as they should, but there are some things to change and finetune still.

This is kind of like a rough sketch of the project...   
In the ready version, there will be everything needed available to use it:   
- Detailed instructions
- Salt states, top file
- Pillars
- Even Vagrant file to create the test environment
- Maybe more...

*This report has been written @ 5.12.2022 and will probably not be updated later.* 

**The real Project is updated, atleast till fully fuctional, and can be found [Here](https://github.com/therealhalonen/domain_control)**
