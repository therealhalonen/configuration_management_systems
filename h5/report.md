# h5 Windows
*Homeworks of fifth class.   
Salt in Windows.   
Getting a slight grip of PowerShell   
Controlling Windows from Linux, with Salt* 

Master/Host:
`I used Debian 11, Lenovo Thinkpad E15`
```bash
~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Debian
Description:	Debian GNU/Linux 11 (bullseye)
Release:	11
Codename:	bullseye
```

For assignments done in Windows (Soon to be Minion):   
**Windows10 Virtualbox VM.
Activated with legit Product Key.**
Network:
```
Adapter1: NAT
Adapter2: Host-only Adapter
```

---
**[Tero Karvinen himself, in the course lecture](https://terokarvinen.com/2022/palvelinten-hallinta-2022p2) and my testings during the class, has been used as a source for these assignments**

### a) Hello Window Salt! Tee Windowsille SLS-tiedostoon Salt-tila, joka tekee tiedoston nimeltä "suolaikkuna.txt".

Now i started installing Salt for Windows:
Downloaded (Currently latest version, written: 26.11.2022):
[**Salt-Minion-3004.2-1-Py3-AMD64.msi**](https://repo.saltproject.io/windows/Salt-Minion-3004.2-1-Py3-AMD64.msi)   

Opened PowerShell with "Run as administrator"
Browsed to the directory, where the downloaded file was:
```powershell
PS C:\Windows\system32> cd C:\Users\antha\Downloads\
PS C:\Users\antha\Downloads>
```

From there i ran the installation command:
```powershell
PS C:\Users\antha\Downloads> msiexec /i Salt-Minion-3004.2-1-Py3-AMD64.msi /norestart MASTER=192.168.56.1 MINION_ID=Win_Minion
```

*Note: My Debian, Host machine was already configured to work as a Master in previous homeworks, assignments and tests.*

So Master is the IP of my Master.
Minion is the name i want for my new Windows Minion.   
After waiting for a while, i closed PowerShell and opened it again.
Now i gave a testrun to see if Salt is working:
```powershell
PS C:\Windows\system32> salt-call --local test.ping
local:
    True
```
**And it was working, atleast locally!**

Next i started the actual assignments and made a new state.    

First i installed `Micro`. Downloaded the latest package for Windows, from:   
https://github.com/zyedidia/micro/releases/    
Extracted to my preferred location and added it to PATH with these instructions:   
https://www.architectryan.com/2018/03/17/add-to-the-path-on-windows-10/   
It is recommended to use safe location, for example something under`C:\User\<username>\` as you shouldn just add all random locations to PATH.

Then i continued from `C:\salt\` which held configs and stuff already.

First a folder, then init.sls
```powershell
PS C:\salt> mkdir hello_winsalt
PS C:\salt> micro .\hello_winsalt\init.sls
```
init.sls:
```yaml
C:\temp\suolaikkuna.txt:
  file.managed:
    - contents: "If you see this, everything went smooth"
```
Now testrun, i thought it wont work, but i tested anyway:
```powershell
PS C:\salt> salt-call --local state.apply hello_winsalt
local:
    Data failed to compile:
----------
    No matching sls found for 'hello_winsalt' in env '
``` 
Just as i thought, state couldnt be found in my `salt env`   
So next good test was to run it with absolute path, to see if its even working as it should:
```bash
PS C:\salt> salt-call --file-root=C:\salt\ --local state.apply hello_winsalt
local:
----------
          ID: C:\temp\suolaikkuna.txt
    Function: file.managed
      Result: True
     Comment: File C:\temp\suolaikkuna.txt updated
     Started: 10:30:16.353720
    Duration: 62.582 ms
     Changes:
              ----------
              diff:
                  New file

Summary for local
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  62.582 ms
```
And yes! State is working.   
But! I want to find my env path.   
I didnt find any info online about the default path in Windows, so i just did it Linux style;   
I made a path i`C:\salt\srv\salt\`  and moved my state folder there.
```powershell
PS C:\salt> mkdir srv/salt


    Directory: C:\salt\srv


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        26.11.2022     10.33                salt


PS C:\salt> mv hello_winsalt .\srv\salt\
```

And now, testrun:    
```powershell
PS C:\salt> salt-call --local state.apply hello_winsalt
local:
----------
          ID: C:\temp\suolaikkuna.txt
    Function: file.managed
      Result: True
     Comment: File C:\temp\suolaikkuna.txt is in the correct state
     Started: 10:35:57.713507
    Duration: 109.426 ms
     Changes:

Summary for local
------------
Succeeded: 1
Failed:    0
------------
Total states run:     1
Total run time: 109.426 ms
```
**An yesss, it was working, path found!**
**Mission accomplished!**   
But! Checked the file is really created, and held the content   
```powershell
PS C:\salt> cat C:\temp\suolaikkuna.txt
If you see this, everything went smooth
```
**Profit!**

*From here, i went straight to an optional assignment:*
### d) Vapaaehtoinen: Ohjaa Windows-konetta Linuxista Saltilla.
**This is it! This is whats its all about! Music to my ears: "Control Windows, with/from Linux"**   

As a continuing straight after the previous part, first thing, was to test if my Windows Minion is already knocking my Master, so i moved to my Debian Host/Master and checked:
```bash
~$ sudo salt-key
Accepted Keys:
minion1
Denied Keys:
Unaccepted Keys:
Rejected Keys:
```
No... Its not there. `minion1` is my Debian Server...
I went back to Windows VM, stopped and started the Salt-Minion with commands:
```powershell
PS C:\Windows\system32> ssm stop salt-minion
salt-minion: STOP: The operation completed successfully.
PS C:\Windows\system32> ssm start salt-minion
salt-minion: START: The operation completed successfully.
```
Also useful debug thing, if this happens, is to stop the daemon, then check:
```powershell
salt-minion -l debug
```
Which shows all the necessary info!   
But i went to my Debian again to check:
```bash
~$ sudo salt-key
Accepted Keys:
minion1
Denied Keys:
Unaccepted Keys:
Win_Minion10
Rejected Keys:
```
An there it was knocking!!!   
I accepted the key, and did a quick ping test, and command test:
```bash
~$ sudo salt-key -a Win_Minion10
The following keys are going to be accepted:
Unaccepted Keys:
Win_Minion10
Proceed? [n/Y] y
Key for minion Win_Minion10 accepted.
~$ sudo salt Win_Minion10 test.ping
Win_Minion10:
    True
~$ sudo salt Win_Minion10 cmd.run "echo Hello World"
Win_Minion10:
    Hello World
```
Everything working!!

---
### b) Ei vihkoa, ei kynää. Kerää Windows-koneen tekniset tiedot tekstitiedostoon. Vapaaehtoinen bonus: Saatko tiedot tallennettua myös json-muodossa?

Now that my Master-Minion was configured for Debian - Windows10 VM, did this from my Debian. To save resources and optimize speed, i left my Windows10 VM running in the backround as `Headless`

As i continued straight from the previous assignment, i already had my Minion responding as it should, so i just ran:
```bash
~$ sudo salt Win_Minion10 grains.items > Win_Minion_Specs
```
This saved all the grains to a file named `Win_Minion_Specs` to my working directory, which could be read with: 
```bash
~$ cat Win_Minion_Specs 
```
As the file is a kilometer long, i will do a short list of info here:
```bash
~$ sudo salt Win_Minion10 grains.item cpu_model num_cpus host id kernelversion master osfullname saltversion > Win_Minion10_shortspecs
~$ cat Win_Minion10_shortspecs 
Win_Minion10:
    ----------
    cpu_model:
        AMD Ryzen 5 5500U with Radeon Graphics         
    host:
        Win10-Virtual
    id:
        Win_Minion10
    kernelversion:
        10.0.19041
    master:
        192.168.56.1
    num_cpus:
        4
    osfullname:
        Microsoft Windows 10 Pro
    saltversion:
        3004.2

```

*Optional bonus, was to make it in JSON format*.   
Used [SaltProject - JSON Out](https://docs.saltproject.io/en/latest/ref/output/all/salt.output.json_out.html) as a source.   
And did it like this:   
```bash
~$ sudo salt Win_Minion10 grains.item cpu_model num_cpus host id kernelversion master osfullname saltversion --out=json > Win_Minion10_shortspecs.json
~$ cat Win_Minion10_shortspecs.json 
{
    "Win_Minion10": {
        "cpu_model": "AMD Ryzen 5 5500U with Radeon Graphics         ",
        "num_cpus": 4,
        "host": "Win10-Virtual",
        "id": "Win_Minion10",
        "kernelversion": "10.0.19041",
        "master": "192.168.56.1",
        "osfullname": "Microsoft Windows 10 Pro",
        "saltversion": "3004.2"
    }
}
```
**Profit!**

---
### c) Kop kop. Onko TCP-portti auki vai kiinni? Näytä esimerkit portin kokeilusta Linuxilla ja Windowsilla. Näytä kummallakin käyttöjärjestelmällä ainakin yksi avoin ja yksi suljettu portti. (Kokeile tätä vain omaan koneeseesi. Vieraiden koneiden ja verkkojen porttiskannaaminen on kiellettyä. Yksittäisen portin testaavat komennot ovat suositeltavia, esim. nc, tnc)

As it states in the assignment;    
**Port scanning is prohibited, atleast in Finland, in networks you dont have the permissions to do so!** 
**These tasks i did in my private network, and to be precise, on my virtual network which doesnt even have a connection to WAN**

For this assignment i used Netcat in my Debian machine.   
I checked the ports needed for Salt and SSH:
```bash
~$ nc -vz 192.168.56.1 4505-4506
Connection to 192.168.56.1 4505 port [tcp/*] succeeded!
Connection to 192.168.56.1 4506 port [tcp/*] succeeded!
~$ nc -vz 192.168.56.1 22
nc: connect to 192.168.56.1 port 22 (tcp) failed: Connection refused

```
Ports for Salts were open, as should.  
Port fo SSH was closed, as should.

As a bonus, i used my preferred port scanner `nmap`:
```bash
~$ nmap 192.168.56.* -p 4505-4506
Starting Nmap 7.80 ( https://nmap.org ) at 2022-11-26 11:43 EET
Nmap scan report for 192.168.56.1
Host is up (0.00013s latency).

PORT     STATE SERVICE
4505/tcp open  unknown
4506/tcp open  unknown

Nmap done: 256 IP addresses (1 host up) scanned in 3.06 seconds
```

Windows part:   
I used PowerShell commands, found and taught by my class colleague, as i have minimal understanding in how PowerShell should work....   
So i checked my Masters ports for Salt and SSH with `test-netconnection` or `tnc` as a short:
```powershell
PS C:\Windows\system32> tnc 192.168.56.1 -p 4505                                                                                                                                                                                                                                                                                                                        ComputerName     : 192.168.56.1
RemoteAddress    : 192.168.56.1
RemotePort       : 4505
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.56.49
TcpTestSucceeded : True



PS C:\Windows\system32> tnc 192.168.56.1 -p 4506


ComputerName     : 192.168.56.1
RemoteAddress    : 192.168.56.1
RemotePort       : 4506
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.56.49
TcpTestSucceeded : True

PS C:\Windows\system32> tnc 192.168.56.1 -p 22
WARNING: TCP connect to (192.168.56.1 : 22) failed


ComputerName           : 192.168.56.1
RemoteAddress          : 192.168.56.1
RemotePort             : 22
InterfaceAlias         : Ethernet 2
SourceAddress          : 192.168.56.49
PingSucceeded          : True
PingReplyDetails (RTT) : 0 ms
TcpTestSucceeded       : False
```
**Salt ports open, SSH closed, as they should!**

Final thoughts:  
I did some tests to scan my Minions ports, and seemed there was none open.   
If i understood correctly, Minions ports doesnt even need to be open, as its the one who connects to Master, so only Master needs to have ports open for succesful interactions.

---
### e) Vapaaehtoinen: Asenna ohjelmia Windowsiin Saltin pkg.installed. Ensin on tämä on toki viriteltävä käyttöön, koska Windowsissa ei ole omaa kattavaa paketinhallintaa.

Now as i did everything needed to control my Windows10 VM from my Debian master, this was pretty straight forward.
As a source i used:  
[Tero Karvinen: Control Windows with Salt](https://terokarvinen.com/2018/control-windows-with-salt/)

Did everything from my Master:   
First installed some winrepo things:
```bash
~$ sudo salt-run winrepo.update_git_repos
https://github.com/saltstack/salt-winrepo-ng.git:
    /srv/salt/win/repo-ng/salt-winrepo-ng
https://github.com/saltstack/salt-winrepo.git:
    /srv/salt/win/repo/salt-winrepo
```
Then updated the database:   
```bash
~$ sudo salt Win_Minion10 pkg.refresh_db
Win_Minion10:
    ----------
    failed:
        0
    success:
        309
    total:
        309
```

Then installed `gedit` to Windows10 VM:
```bash
~$ sudo salt Win_Minion10 pkg.install gedit
Win_Minion10:
    ----------
    gedit:
        ----------
        new:
            2.30.1
        old:
```
Worked and to verify, checked my Windows10 VM:   
![Image1](https://github.com/therealhalonen/configuration_management_systems/blob/master/h5/res/gedit_win.png)   
**Worked!**

To see all the packages available from `pkg.install` after these steps, go:   
https://github.com/saltstack/salt-winrepo-ng
It seems to be actively updated.

As a additional bonus i tried [Chocolatey](https://community.chocolatey.org/packages) , which i havent heard of before. But still using mentioned Tero's guide as a source, i installed few things as a test:   

First i needed to install Chocolatey
```
~$ sudo salt Win_Minion10 pkg.install chocolatey
Win_Minion10:
    ----------
    chocolatey:
        ----------
        new:
            1.2.0
        old:
```

Then installed package from Chocolatey.   
I used `Notepad++` as a test, which was one of my favorites for Windows, back in the days.  
```bash
~$ sudo salt Win_Minion10 chocolatey.install notepadplusplus
Win_Minion10:
    Chocolatey v1.2.0
    Installing the following packages:
    notepadplusplus
    By installing, you accept licenses for the packages.
    
    chocolatey-compatibility.extension v1.0.0 [Approved]
    chocolatey-compatibility.extension package files install completed. Performing other installation steps.
     Installed/updated chocolatey-compatibility extensions.
     The install of chocolatey-compatibility.extension was successful.
      Software installed to 'C:\ProgramData\Chocolatey\extensions\chocolatey-compatibility'
    
    chocolatey-core.extension v1.4.0 [Approved]
    chocolatey-core.extension package files install completed. Performing other installation steps.
     Installed/updated chocolatey-core extensions.
     The install of chocolatey-core.extension was successful.
      Software installed to 'C:\ProgramData\Chocolatey\extensions\chocolatey-core'
    
    notepadplusplus.install v8.4.7 [Approved]
    notepadplusplus.install package files install completed. Performing other installation steps.
    Installing 64-bit notepadplusplus.install...
    notepadplusplus.install has been installed.
    WARNING: No registry key found based on  'Notepad\+\+*'
    notepadplusplus.install installed to 'C:\Program Files\Notepad++'
    Added C:\ProgramData\Chocolatey\bin\notepad++.exe shim pointed to 'c:\program files\notepad++\notepad++.exe'.
      notepadplusplus.install can be automatically uninstalled.
     The install of notepadplusplus.install was successful.
      Software installed as 'exe', install location is likely default.
    
    notepadplusplus v8.4.7 [Approved]
    notepadplusplus package files install completed. Performing other installation steps.
     The install of notepadplusplus was successful.
      Software install location not explicitly set, it could be in package or
      default install location of installer.
    
    Chocolatey installed 4/4 packages. 
     See the log for details (C:\ProgramData\Chocolatey\logs\chocolatey.log).

```
**Done**  
And to verify:   
![Image2](https://github.com/therealhalonen/configuration_management_systems/blob/master/h5/res/notepadplus_win.png)   
**Worked!**

**If i understood correctly, i now have the ability to install little less then 10k packages to my Windows Minion now!!!**

*The Power of the Sun, in the Palm of My Hand*  - Otto Octavius, Spiderman 2.

---
These last, optional assingments, il probably test later, without reporting:
```bash
f) Vapaaehtoinen: Kokeile Ansiblea.
g) Vapaaehtoinen: Kokeile Terraformia.
```
Mostly i just read documentations, to figure out how they work and why:   
[Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)   
[Terraform](https://www.terraform.io/)

---
