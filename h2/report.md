# h2 Package-File-Service
*Homeworks of second class. 
Getting inside the master-minion architecture, Saltstack fundamentals and configuring daemons with `salt`.*

`I used Debian 11, Lenovo Thinkpad E15`
```bash
~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Debian
Description:	Debian GNU/Linux 11 (bullseye)
Release:	11
Codename:	bullseye
```

---
### x) Lue ja tiivistä

-  [Karvinen 2018: Salt Quickstart - Salt Stack Master and Slave on Ubuntu Linux](http://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux)
	- To have a working Master-Minion architecture:
		- You first install `salt-master` on a computer, then you install `salt-minion` to another computer. 
		- But thats not all. 
			- You tell the Minion, whos the boss, by editing `/etc/salt/minion` and adding your Masters address `master: <ip_of_master>`.  
			- Then the Master will decide if it will accept the Minion, by checking `sudo salt-key` . 
			- This is where it sees the Minion knocking, and will either accept it with `sudo salt-key -a <minion>` or delete it with `sudo salt-key -d <minion>` . 
		- Now it can call the Minion back, to identify itself with `sudo salt '*' cmd.run 'whoami`.
	
- Salt official documentation: [Salt Getting Started Guide-kirjasta luvut Understanding SaltStack ja SaltStack Fundamentals ja SaltStack Configuration Management: Functions](https://docs.saltstack.com/en/getstarted/)
	- SaltStack is a lightweight, scalable Python-based software that you can use to config 10-10000 systems in seconds. 1 Master -> Several Minions to command.
	- Article gives a reader a good informations about SaltStack, how it works, how its configured and how you can configure it yourself.
	- Vagrant is a software that lets you  create and run several Virtualbox VM:s in the background.  It takes away the pain to create each Minion -machine separately.
	- About Salt states: Salt includes Jinja2 templating engine which can be used to manage them and other files. With Jinja2 you can make loops, conditional statements and use it to call Salt to get real-time data from the system.

**Articles above and [Tero Karvinen himself, in the course lecture](https://terokarvinen.com/2022/palvelinten-hallinta-2022p2) , has been used as a source for this homework.**

---
### a) Demonin asetukset. Säädä jokin demoni (asenna+tee asetukset+testaa) package-file-service -rakenteella. Ensin käsin: muista tehdä ja raportoida asennus ensin käsin, vasta sitten automatisoiden. Jos osaat hyvin, voit tehdä jonkin eri asetuksen kuin tunnilla. Harjoitusta varten tulee siis tehdä alusta ja raportoida samalla.

To this assignment, i had an already installed and configured Debian 11 (bullseye) virtualbox, virtual machine.

**Configuring a daemon:**

I used `apache2`, as its the one, im pretty familiar with.   
Installed apache2:
```bash
~$ sudo apt-get update && sudo apt-get install apache2
```

Next i tested if the daemon is running:
```bash
~$ sudo systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor prese>
     Active: active (running) since Sun 2022-11-06 16:26:02 EET; 3min 20s ago
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 2708 (apache2)
      Tasks: 55 (limit: 2277)
     Memory: 8.9M
        CPU: 34ms
     CGroup: /system.slice/apache2.service
             ├─2708 /usr/sbin/apache2 -k start
             ├─2710 /usr/sbin/apache2 -k start
             └─2711 /usr/sbin/apache2 -k start

```
**And it was.**

Then, for not printing the whole page i used `curl` and checked only `headers`with parameter `-I` , to see if the default page is showing as it should:
```bash
~$ curl -I localhost
HTTP/1.1 200 OK
Date: Sun, 06 Nov 2022 14:38:46 GMT
Server: Apache/2.4.54 (Debian)
Last-Modified: Sun, 06 Nov 2022 14:26:00 GMT
ETag: "29cd-5ecce16059dc2"
Accept-Ranges: bytes
Content-Length: 10701
Vary: Accept-Encoding
Content-Type: text/html
```

Next i changed the default page, to my "personal" testpage.
First i checked if i remember the path correctly
```bash
~$ ls /var/www/html
index.html
```
- Check

Made a backup of the default `.html` file:

```bash
~$ cd /var/www/html
/var/www/html$ sudo cp index.html index.html.bak
```

Then wrote on top of the `index.html`
```bash
/var/www/html$ echo -e 'Hello World!\nThis is your front page'|sudo tee  index.html
Hello World!
This is your front page
/var/www/html$ cat index.html
Hello World!
This is your front page
```

And tested if it was working:
```bash
/var/www/html$ curl localhost
Hello World!
This is your front page
```
So the `localhost` is the address of my apache2 server AKA my pc.   
*localhost = /var/www/html/index.html*

Next i configured `userdir`:
Went to my `home`, made `public_html` folder, made `index.html` to `public_html`.    
Enabled `apache2` `userdir` module, and restarted the `apache2` daemon.

```bash
/var/www/html$ cd ~
~$ mkdir public_html
~$ echo 'This is my personal space!' > public_html/index.html
~$ sudo a2enmod userdir
Enabling module userdir.
To activate the new configuration, you need to run:
  systemctl restart apache2
~$ sudo systemctl restart apache2
```

Now testrun to see if it shows:
```bash
~$ curl localhost/~<username>/
This is my personal space!
```
Working!

Now i had configured my Apache2, on my soon to be Master, manually. Next up, was configuring it to minion, with `salt`.

**Configuring a daemon with `salt` , from Master to Minion**:

First i installed `salt-master`:
***NOTE!
If using Ubuntu as a minion and Debian as a Master, like i did, they install different version (Debian has older) from the default repo.
So for Debian i installed the latest with instructions from [SaltProject](https://repo.saltproject.io/#debian):***

```bash
# Download key
sudo curl -fsSL -o /usr/share/keyrings/salt-archive-keyring.gpg https://repo.saltproject.io/py3/debian/11/amd64/3004/salt-archive-keyring.gpg
# Create apt sources list file
echo "deb [signed-by=/usr/share/keyrings/salt-archive-keyring.gpg arch=amd64] https://repo.saltproject.io/py3/debian/11/amd64/3004 bullseye main" | sudo tee /etc/apt/sources.list.d/salt.list

~$ sudo apt-get update && install salt-master
```

Then checked, if the daemon was running automatically after the installation:
```bash
~$ sudo systemctl status salt-master
● salt-master.service - The Salt Master Server
     Loaded: loaded (/lib/systemd/system/salt-master.service; enabled; vendor p>
     Active: active (running) since Sun 2022-11-06 17:03:34 EET; 25min ago
       Docs: man:salt-master(1)
             file:///usr/share/doc/salt/html/contents.html
             https://docs.saltstack.com/en/latest/contents.html
   Main PID: 703 (salt-master)
      Tasks: 32 (limit: 2277)
     Memory: 252.9M
        CPU: 13.311s
     CGroup: /system.slice/salt-master.service
             ├─ 703 /usr/bin/python3 /usr/bin/salt-master
```
**Working!**

Next i started to configure my `minion` by installing a Ubuntu Server 22.10 virtual machine, as its pretty lightweight.     
When i made the machine, i selected a Nat Network, which i had DHCP configured from Virtualbox preferences, ready to be used.   
![Image1][https://github.com/therealhalonen/configuration_management_systems/blob/master/h2/res/Nat_Network.png]]
This `HaloNetwork` is also what my Master uses.

**Nothing special needs to be made during the installation of Ubuntu Server.** 

After the server is ready and booted, i updated the packages and stuff, but not going to through all the details here, as we go straight to installing `salt-minion`

Master:
```bash
~$ hostname -I
10.0.2.15 
```
IP is needed for minion.

Ubuntu Server (soon to be minion):
```bash
~$ sudo apt-get update && sudo apt-get install salt-minion
```

Telling whos the boss and whos calling:
```bash
~$ echo -e 'master: 10.0.2.15\nid: humble_servant'|sudo tee /etc/salt/minion
master: 10.0.2.15
id: humble_servant 
```

Kicking the daemon:
```bash
~$ sudo systemctl restart salt-minion
```

Then went back to my Master, to see if my Minion is knocking:
```bash
~$ sudo salt-key
Accepted Keys:
Denied Keys:
Unaccepted Keys:
humble_servant
Rejected Keys:
```
Yesss, its working. 

Now i accepted the key:
```bash
~$ sudo salt-key -a humble_servant
The following keys are going to be accepted:
Unaccepted Keys:
humble_servant
Proceed? [n/Y] y
Key for minion humble_servant accepted.
```

Then, as always, i wanted to test if my new minion is responding:
```bash
~$ sudo salt humble_servant test.ping
humble_servant:
    True
```
**Working!**

**Now i wanted to configure it automatically to my Minion:** 
*( Also: Installing Master and Minion to different machines)*

*Working in my Master:*
First i made the correct folder and and started to edit `init.sls`
```bash
~$ sudo mkdir -p /srv/salt/apache2/
~$ sudoedit -p /srv/salt/apache2/init.sls
```

Also copied my index.html and public index.html to the apache2 state directory:
```bash
~$ sudo cp public_html/index.html /srv/salt/apache2/public.html
~$ sudo cp /var/www/html/index.html /srv/salt/apache2/index.html
```

I wanted my Minion to host the same webpage as my Master.  
I also wanted it to have default personal page in home folder, forced by Master (`~/public_html/index.html)

```python
#init.sls
apache2:
  pkg.installed

/var/www/html/index.html:
  file.managed:
    - source: "salt://apache2/index.html"

{% set script = salt['cmd.script']('salt://apache2/userlist.sh') %}
{% set list = script.stdout.split('\n') %}
{% for user in ( list ) %}
~/{{ user }}/public_html/index.html:
  file.managed:
    - source: "salt://apache2/public.html"
    - makedirs: True
    - user: {{ user }}
    - group: {{ user }}
    - mode: 0744
{% endfor %}

/etc/apache2/mods-enabled/userdir.conf:
  file.symlink:
    - target: ../mods-available/userdir.conf

/etc/apache2/mods-enabled/userdir.load:
  file.symlink:
    - target: ../mods-available/userdir.load

apache2.service:
  service.running:
    - watch: 
      - file: /etc/apache2/mods-enabled/userdir.load
      - file: /etc/apache2/mods-enabled/userdir.conf
``` 
Sources and help i got for this `init.sls`:
- [Jinja Documentation](https://jinja.palletsprojects.com/en/3.1.x/) 
- [Github - Jinja Issues](https://github.com/pallets/jinja/issues/898) -
- [StackOverflow - Read next line in Jinja2](https://stackoverflow.com/questions/38486452/read-next-line-in-jinja2)
- [Jinja2 for loops](https://ttl255.com/jinja2-tutorial-part-2-loops-and-conditionals/)
- [Stack Overflow - Splitting in Jinja](https://stackoverflow.com/questions/30515456/split-string-into-list-in-jinja)
- [Stack Overflow- stdout to file](https://stackoverflow.com/a/69565451)
- [SaltProject - State Tutorials](https://docs.saltproject.io/en/latest/topics/tutorials/starting_states.html)
- [Github - SaltStack - modify users home](https://github.com/saltstack/salt/issues/35594)
- [SaltProject - remote execution](https://docs.saltstack.cn/topics/execution/remote_execution.html)   
  **Special thanks: [Niko Heiskanen himself](https://heiskanen.rocks/)**

**And to break it down:**
1. It installs/check if already installed apache2
2. It copies the index.html from Masters apache2 state folder to Minions default page location
3. It runs a script from Masters apache2 state folder to Minion, and gets a list of users. Then it uses the list to copy the default userdir public.html to every users `~/public_html/index.html`. If the path doesnt exist in Minion, it creates it.
4. It enables `userdir` by symlinking the needed files between each other
5. It kicks the apache2 daemon, if its not running.

**This was a bonus stuff for myself, but:
As im quite familiar with /etc/passwd file structure, i made it to print all the users that have home, then piped it to cut from the `:` to print only the first word = username.   
After that i figured theres an easier way... With `ls /home/` shows directories that really exist.   
Again, this was done with learning in mind, for how to manipulate several users home directories, without defining a specific user manually.**
```bash
cat /etc/passwd|grep home|cut -d ":" -f1
```
Gets the list of user that have `home`.

Or better, the whole script made easier:
```bash
#userlist
#!/bin/bash
for USER in $(ls /home/);
do
    echo $USER
done
```
Permissions for the script: `chmod 0700 userlist`

Here is my apache2 state folder:
```bash
~$ ls /srv/salt/apache2
index.html  init.sls  public.html  userlist
```

Then i tested if the setup was working ( i had 2 users on my Minion: user and testuser):
```bash
sudo salt humble_servant state.apply apache2
```

```bash
~$ sudo salt humble_servant state.apply apache2
humble_servant:
----------
          ID: apache2
    Function: pkg.installed
      Result: True
     Comment: The following packages were installed/updated: apache2
     Started: 12:20:49.485491
    Duration: 11514.88 ms
     Changes:   
              ----------
              apache2:
                  ----------
                  new:
                      2.4.52-1ubuntu4.1
                  old:
----------
          ID: /var/www/html/index.html
    Function: file.managed
      Result: True
     Comment: File /var/www/html/index.html updated
     Started: 12:21:01.004557
    Duration: 42.47 ms
     Changes:   
              ----------
              diff:
                  --- 
                  +++ 
                  @@ -1,363 +1 @@
				# HERE WAS ALL THE APACHE2 DEFAULT INDEX.HTML CONTENT 
                  +Hello World!
                  +This is your front page
----------
          ID: /home/testuser/public_html/index.html
    Function: file.managed
      Result: True
     Comment: File /home/testuser/public_html/index.html updated
     Started: 12:21:01.047154
    Duration: 18.039 ms
     Changes:   
              ----------
              diff:
                  New file
              group:
                  testuser
              mode:
                  0744
              user:
                  testuser
----------
          ID: /home/user/public_html/index.html
    Function: file.managed
      Result: True
     Comment: File /home/user/public_html/index.html updated
     Started: 12:21:01.065310
    Duration: 14.001 ms
     Changes:   
              ----------
              diff:
                  New file
              group:
                  user
              mode:
                  0744
              user:
                  user
----------
          ID: /etc/apache2/mods-enabled/userdir.conf
    Function: file.symlink
      Result: True
     Comment: Created new symlink /etc/apache2/mods-enabled/userdir.conf -> ../mods-available/userdir.conf
     Started: 12:21:01.079912
    Duration: 3.858 ms
     Changes:   
              ----------
              new:
                  /etc/apache2/mods-enabled/userdir.conf
----------
          ID: /etc/apache2/mods-enabled/userdir.load
    Function: file.symlink
      Result: True
     Comment: Created new symlink /etc/apache2/mods-enabled/userdir.load -> ../mods-available/userdir.load
     Started: 12:21:01.083885
    Duration: 1.204 ms
     Changes:   
              ----------
              new:
                  /etc/apache2/mods-enabled/userdir.load
----------
          ID: apache2.service
    Function: service.running
      Result: True
     Comment: Service restarted
     Started: 12:21:01.113934
    Duration: 1079.308 ms
     Changes:   
              ----------
              apache2.service:
                  True

Summary for humble_servant
------------
Succeeded: 7 (changed=7)
Failed:    0
------------
Total states run:     7
Total run time:  12.674 s
~$ 
```
**Working**. And to verify, i went to my Server, and checked the pages:
```bash
~$ curl localhost
Hello World!
This is your front page
~$ curl localhost/~user/
This is a default front page, of users own space.
```
And again, **working!** I also tested to shut the apache2 from my Serve to see if it kicks back.
```bash
~$ sudo systemctl stop apache2
```

And from my Master:
```bash
~$ sudo salt humble_servant state.apply apache2

          ID: apache2.service
    Function: service.running
      Result: True
     Comment: Started service apache2.service
     Started: 12:37:45.449296
    Duration: 108.376 ms
     Changes:   
              ----------
              apache2.service:
                  True

```
Done!

---
### b) Asenna Salt herra-orja niin, että molemmat ovat samalla koneella. Voit tehdä ne esimerkiksi uudelle virtuaalikoneelle, niin pääset kokeilemaan puhtaalta pöydältä.

I started this assignment, by installing a fresh new Debian 11 virtual machine to my virtualbox. 
Theres no special tricks during the installation, but this is a nice ive guide ive used before: [Tero Karvinen: Install Debian on Virtualbox](https://terokarvinen.com/2021/install-debian-on-virtualbox/)

Note: hostname is `homework`
After installation procedure was finished and system was booted and operational, i started with updating the packages and installing updates.

```bash
~$ sudo apt-get update && sudo apt-get dist-upgrade
```

After updates and a reboot, i started to install `salt-master` and `salt minion`.
```bash
~$ sudo apt-get install salt-master salt-minion
```

After installing, i checked that they were both running:

Minion:
```bash
~$ sudo systemctl status salt-minion
● salt-minion.service - The Salt Minion
     Loaded: loaded (/lib/systemd/system/salt-minion.service; enabled; vendor p>
     Active: active (running) since Sun 2022-11-06 11:02:54 EET; 19s ago
       Docs: man:salt-minion(1)
             file:///usr/share/doc/salt/html/contents.html
             https://docs.saltstack.com/en/latest/contents.html
   Main PID: 2518 (salt-minion)
      Tasks: 4 (limit: 2277)
     Memory: 75.8M
        CPU: 786ms
     CGroup: /system.slice/salt-minion.service
```

Master:
```bash
~$ sudo systemctl status salt-master
● salt-master.service - The Salt Master Server
     Loaded: loaded (/lib/systemd/system/salt-master.service; enabled; vendor p>
     Active: active (running) since Sun 2022-11-06 11:02:56 EET; 31s ago
       Docs: man:salt-master(1)
             file:///usr/share/doc/salt/html/contents.html
             https://docs.saltstack.com/en/latest/contents.html
   Main PID: 2678 (salt-master)
      Tasks: 32 (limit: 2277)
     Memory: 197.7M
        CPU: 5.515
     CGroup: /system.slice/salt-master.service    
```
All good so far, both daemons are active.

First i needed the masters IP:
```bash
~$ hostname -I
10.0.2.15 
```

Then i configured the minion, so i added a masters IP to minions config:
```bash
~$ sudoedit /etc/salt/minion
```

```bash
#minion
master: 10.0.2.15
```
Save and exit.

Then i restarted the minion daemon:
```bash
~$ sudo systemctl restart salt-minion
```

Next up, was accepting the minion key(s).
So i checked if it was showing in:
```bash
~$ sudo salt-key
Accepted Keys:
Denied Keys:
Unaccepted Keys:
homework
Rejected Keys:
```

Outcome is, as it should, because it shouldnt be accepted without me.

Accepted the key:
```bash
~$ sudo salt-key -a homework
The following keys are going to be accepted:
Unaccepted Keys:
homework
Proceed? [n/Y] 
Key for minion homework accepted.
```

Now that the `homework` minion key was accepted, i tested if the setup was working by pinging the minion with my master:
```bash
~$ sudo salt '*' test.ping
homework:
    True
```
**Ping was answered and slave-minion setup in same computer was working as it should.**

---
### c) Aja jokin tila paikallisesti ilman master-slave arkkitehtuuria. Analysoi debug-tulostetta. 'sudo salt-call --local state.apply hellotero -l debug'

Used: **Debian 11 - virtual machine**

Started by installing only `salt-master`
```bash
~$ sudo apt-get update && sudo apt-get install salt-master
```

Then made the folder for salt states:
```bash
$ sudo mkdir -p /srv/salt/
```

And moving there:
```bash
$ cd /srv/salt/
```

Next i started my "Hello World" testings, by making a folder and an init file inside it.
```bash
/srv/salt$ sudo mdir hello
/srv/salt$ sudoedit hello/init.sls
```

Inside the file:
```yaml
# init.sls
/tmp/helloworld:
  file.managed:
    - contents: "Hello World"
```
Save & Quit. 

Then it was time to inspect and analyze the debug log.   
I didnt test if my file was correctly made or anything, as i would see errors anyway when trying to run it.   
So:
```bash
~$ sudo salt-call --local state.apply hello -l debug
```

Debug log:
```bash
DEBUG   ] Missing configuration file: /etc/salt/minion
[DEBUG   ] Guessing ID. The id can be explicitly set in /etc/salt/minion
[DEBUG   ] Found minion id from generate_minion_id(): homework
[DEBUG   ] Configuration file path: /etc/salt/minion
[WARNING ] Insecure logging configuration detected! Sensitive data may be logged.
[DEBUG   ] Grains refresh requested. Refreshing grains.
[DEBUG   ] Missing configuration file: /etc/salt/minion
[DEBUG   ] Elapsed time getting FQDNs: 0.019656658172607422 seconds
[DEBUG   ] LazyLoaded zfs.is_supported
[DEBUG   ] Determining pillar cache
[DEBUG   ] LazyLoaded jinja.render
[DEBUG   ] LazyLoaded yaml.render
[DEBUG   ] LazyLoaded jinja.render
[DEBUG   ] LazyLoaded yaml.render
[DEBUG   ] LazyLoaded state.apply
[DEBUG   ] LazyLoaded direct_call.execute
[DEBUG   ] LazyLoaded saltutil.is_running
[DEBUG   ] LazyLoaded grains.get
[DEBUG   ] LazyLoaded config.get
[DEBUG   ] LazyLoaded roots.envs
[DEBUG   ] Could not LazyLoad roots.init: 'roots.init' is not available.
[DEBUG   ] Updating roots fileserver cache
[DEBUG   ] Gathering pillar data for state run
[DEBUG   ] Determining pillar cache
[DEBUG   ] LazyLoaded jinja.render
[DEBUG   ] LazyLoaded yaml.render
[DEBUG   ] Finished gathering pillar data for state run
[INFO    ] Loading fresh modules for state activity
[DEBUG   ] LazyLoaded jinja.render
[DEBUG   ] LazyLoaded yaml.render
[DEBUG   ] Could not find file 'salt://hello.sls' in saltenv 'base'
[DEBUG   ] In saltenv 'base', looking at rel_path 'hello/init.sls' to resolve 'salt://hello/init.sls'
[DEBUG   ] In saltenv 'base', ** considering ** path '/var/cache/salt/minion/files/base/hello/init.sls' to resolve 'salt://hello/init.sls'
[DEBUG   ] compile template: /var/cache/salt/minion/files/base/hello/init.sls
[DEBUG   ] Jinja search path: ['/var/cache/salt/minion/files/base']
[DEBUG   ] LazyLoaded roots.envs
[DEBUG   ] Could not LazyLoad roots.init: 'roots.init' is not available.
[PROFILE ] Time (in seconds) to render '/var/cache/salt/minion/files/base/hello/init.sls' using 'jinja' renderer: 0.016947507858276367
[DEBUG   ] Rendered data from file: /var/cache/salt/minion/files/base/hello/init.sls:
/tmp/helloworld:
  file.managed:
    - contents: "Hello World"

[DEBUG   ] Results of YAML rendering: 
OrderedDict([('/tmp/helloworld', OrderedDict([('file.managed', [OrderedDict([('contents', 'Hello World')])])]))])
[PROFILE ] Time (in seconds) to render '/var/cache/salt/minion/files/base/hello/init.sls' using 'yaml' renderer: 0.0002498626708984375
[DEBUG   ] LazyLoaded config.option
[DEBUG   ] LazyLoaded file.managed
[INFO    ] Running state [/tmp/helloworld] at time 15:41:07.427333
[INFO    ] Executing state file.managed for [/tmp/helloworld]
[DEBUG   ] LazyLoaded file.source_list
[INFO    ] File changed:
New file
[INFO    ] Completed state [/tmp/helloworld] at time 15:41:07.437698 (duration_in_ms=10.365)
[DEBUG   ] File /var/cache/salt/minion/accumulator/139699425179056 does not exist, no need to cleanup
[DEBUG   ] LazyLoaded state.check_result
[DEBUG   ] LazyLoaded highstate.output
[DEBUG   ] LazyLoaded nested.output
local:
----------
          ID: /tmp/helloworld
    Function: file.managed
      Result: True
     Comment: File /tmp/helloworld updated
     Started: 15:41:07.427333
    Duration: 10.365 ms
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
Total run time:  10.365 ms
```

File was created, and to verify, i checked:
```
~$ ls /tmp/hello*
/tmp/helloworld
~$ cat /tmp/helloworld
Hello World
```

Working! Now i started to analyze the log.
- I saw it notifies that minion configuration file is missing.  Of course it is, as i dont have any minions. 
- It tries to guess the minion id, and finds it with/in some function `generate_minion_id()` . That might be, as i have tested a minion in my system before, and deleted them, but it may also use my localhost name for that. 
- It finds the file `salt://hello/init.sls` which i made, uses it to compile and render the data using `yaml`
- It informs the results of rendering.
- It cleans up afterwords, and informs that some file doesnt exist, so it doesnt need to be cleaned up.
- It also informs timestamps and how long it took to do some steps.


### d) Vapaaehtoinen: Asenna Salt herra ja orja eri koneille.
- This i did during the [**a)**](https://github.com/therealhalonen/new_repo/edit/master/readme.md#a-demonin-asetukset-s%C3%A4%C3%A4d%C3%A4-jokin-demoni-asennatee-asetuksettestaa-package-file-service--rakenteella-ensin-k%C3%A4sin-muista-tehd%C3%A4-ja-raportoida-asennus-ensin-k%C3%A4sin-vasta-sitten-automatisoiden-jos-osaat-hyvin-voit-tehd%C3%A4-jonkin-eri-asetuksen-kuin-tunnilla-harjoitusta-varten-tulee-siis-tehd%C3%A4-alusta-ja-raportoida-samalla) part.
