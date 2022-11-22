# h4 Omat komennot
*Homeworks of fourth class.   
How scripting is done right.   
SaltStack: how to copy scripts or a directory full of scripts, to minions!   
Checking previously made projects related to SaltStack and trying some modules other people have made*

Master:
`I used Debian 11, Lenovo Thinkpad E15`
```bash
~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Debian
Description:	Debian GNU/Linux 11 (bullseye)
Release:	11
Codename:	bullseye
```
Minion:
I usually apply and test everything in my `minion1` which is mentioned several times in these reports and assignments.   
That is a one ive configured in [h3 - Vagrant part](https://github.com/therealhalonen/configuration_management_systems/blob/master/h3/report.md#vagrant-with-salt).   
Working directory to control my minion:
```bash
~/Projects/vagranttest
```
This is just to clarify why there are stuff done sometimes in different directories, atleast in this report.
```bash
vagrant@minion1:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Debian
Description:	Debian GNU/Linux 11 (bullseye)
Release:	11
Codename:	bullseye
```

---
**[Tero Karvinen himself, in the course lecture](https://terokarvinen.com/2022/palvelinten-hallinta-2022p2) and my hazard testings before, has been used as a source for script stuff.**

*Due to schedules, i started these assignments, with very simple scripts, that will definitely work and i might add more complex stuff later.*

### a) Hei komento! Tee järjestelmään uusi "hei maailma" -komento ja asenna se orjille Saltilla. Liitä raporttiisi orjan 'ls -l /usr/local/bin/' tulosteesta ainakin se rivi, jolla näkyy uuden komentotiedostosi oikeudet.

Hellp World, my old friend.   
This one i started with a nice oneliner, to create the file, add content, fix permissions, testrun and print the info of the file.   
I do not recommend this style, but did it for the challenge and kicks!   
```bash
~$ SCRIPT="/usr/local/bin/hello_world"; echo -e '#!/bin/bash\necho "Hello World"'|sudo tee $SCRIPT && sudo chmod 755 $SCRIPT && echo 'Script output:' && hello_world && echo Permissions:&& ls -la $SCRIPT
#!/bin/bash
echo "Hello World"
Script output:
Hello World
Permissions:
-rwxr-xr-x 1 root root 31 18.11. 16:40 /usr/local/bin/hello_world
```
That was a messy looking one...    
Now to get it to the minions, i used my trusted minion aka `minion1` - a Debian 11 server, which i already had setup previously, to take my orders.   
First i made a new state, to my existing `/srv/salt`   
```bash
~$ cd /srv/salt/
/srv/salt$ sudo mkdir test_script
/srv/salt$ sudoedit scripts/test_script/init.sls
```
Inside init.sls:   
```bash
# init.sls
/usr/local/bin/hello_world:
  file.managed:
    - source: "salt://test_script/hello_world"
    - mode: '0755'
```
Next i copied the hello_world script to my salt state folder which i defined in the init.sls, and verified the folder content:
```bash
/srv/salt$ sudo cp /usr/local/bin/hello_world test_script/
/srv/salt$ ls test_script/
hello_world  init.sls
```

State apply:   
```bash
/srv/salt$ sudo salt minion1 state.apply test_script
minion1:
----------
          ID: /usr/local/bin/hello_world
    Function: file.managed
      Result: True
     Comment: File /usr/local/bin/hello_world updated
     Started: 14:54:28.055687
    Duration: 34.526 ms
     Changes:   
              ----------
              diff:
                  New file
              mode:
                  0755

Summary for minion1
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  34.526 ms
```
Everything looking as it should, but to verify, i went in with ssh to check that it runs and verify the permissions.
```bash
vagrant@minion1:~$ hello_world
Hello World
vagrant@minion1:~$ ls -la /usr/local/bin/hello_world
-rwxr-xr-x 1 root root 31 Nov 18 14:54 /usr/local/bin/hello_world
```
**Done!**

**Bonus, for the permission check!**   
Do it with salt, from Master to Minion, with `cmd.run`:
```bash
/srv/salt$ sudo salt minion1 cmd.run 'ls -la /usr/local/bin/hello_world'
minion1:
    -rwxr-xr-x 1 root root 31 Nov 18 14:54 /usr/local/bin/hello_world
```
**Sweeeeet!!!**

---
### b) whatsup.sh. Tee järjestelmään uusi komento, joka kertoo ajankohtaisia tietoja; asenna se orjille. Vinkkejä: Voit näyttää valintasi mukaan esimerkiksi päivämäärää, säätä, tietoja koneesta, verkon tilanteesta...

For this, i had an idea! And as i wrote this, i tried to make it happen...   
A script, with a variable, that displays the weather of the defined variable when ran.   
If no variable given, it would display the current/nearest location, which i think means the one that the system is configured...   
But lets get it cookin! This kind, ive never done before.   
As a source, i used:
- https://wttr.in/   

Which works as it is with `curl wttr.in/<city of your choice>`, or blank for one defined by your system   
But script is much funner!
I made a file weather with content:   
```bash
#!/bin/bash
# If variable is empty:
if [ -z "$1" ]
then
      curl wttr.in
# If variable has content:
else
      curl wttr.in/$1
fi
```
Gave it the permissions for me, group and others:   
```bash
~$ chmod 755 weather
~$ ls -la weather
-rwxr-xr-x 1 sicki sicki 132 18.11. 17:21 weather
```
Testrun without a variable:   
```bash
~$ ./weather
Weather report: Helsinki, Finland

     \  /       Partly cloudy
   _ /"".-.     -1(-8) °C      
     \_(   ).   ↙ 19 km/h      
     /(___(__)  10 km          
                0.0 mm
```
**Muahahah!!!!! Its alive!!!!**   
And then New York:
```bash
~$ ./weather New York
Weather report: New

      \   /     Sunny
       .-.      +8(5) °C       
    ― (   ) ―   ↙ 26 km/h      
       `-’      16 km          
      /   \     0.0 mm
```
Working!   
And lastly Korvatunturi as a bonus:   
```bash
~$ ./weather Korvatunturi
Weather report: Korvatunturi

                Mist
   _ - _ - _ -  -1(-3) °C      
    _ - _ - _   ↖ 4 km/h       
   _ - _ - _ -  2 km           
                0.0 mm   
```
**DONE!** That was nice.   
*Note: The prints were/are alot longer and contains forecasts for 3 days, but i cut em to save space.*

To get them to minions, i again, made a new state:
```bash
/srv/salt$ sudo mkdir weather
/srv/salt$ sudoedit weather/init.sls
/srv/salt$ sudo cp ~/weather weather/
```
init.sls:
```bash
/usr/local/bin/weather:
  file.managed:
    - source: salt://weather/weather
    - mode: '0755'

# curl is also needed, so added just in case
curl:
  pkg.installed
```

Then test run:
```bash
/srv/salt$ sudo salt minion1 state.apply weather
minion1:
----------
          ID: /usr/local/bin/weather
    Function: file.managed
      Result: True
     Comment: File /usr/local/bin/weather is in the correct state
     Started: 15:43:19.775329
    Duration: 37.611 ms
     Changes:   
----------
          ID: curl
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 15:43:20.532797
    Duration: 18.124 ms
     Changes:   

Summary for minion1
------------
Succeeded: 2
Failed:    0
------------
Total states run:     2
Total run time:  55.735 ms
```
Looking good, and this time i went in again with SSH and checked if it works:   
```bash
~/Projects/vagranttest$ vagrant ssh minion1
vagrant@minion1:~$ weather Espoo
Weather report: Espoo

     \  /       Partly cloudy
   _ /"".-.     -1(-8) °C      
     \_(   ).   ↙ 19 km/h      
     /(___(__)  10 km          
                0.0 mm 
```
**All good and working!!**   

--- 
### c) hello.py. Tee järjestelmään uusi komento Pythonilla ja asenna se orjille. Vinkkejä: Hei maailma riittää, mutta propellihatut saavat toki koodaillakin. Shebang on "#!/usr/bin/python3". Helpoin Python-komento on: print("Hei Tero!")

For this i went the easy way.   
Made a `hello` script which takes a variable, and if no variable given, it prints `Hello World`
Script:
```
~$ micro pyhello
```

```python
#!/usr/bin/python3
import sys

if len(sys.argv) > 1:
	print(f"Hello {sys.argv[1]}")
else:
    print("Hello World")
```
Again permissions and testruns:
```bash
~$ chmod 755 pyhello
~$ ./pyhello You
Hello You
~$ ./pyhello
Hello World
```
Working like it should!   
Off to the minion again, he/she goes wuhuuu!   
First statefolder, init.sls and my new fresh `pyhello` to go.   
```bash
~$ cd /srv/salt
/srv/salt$ sudo mkdir pyscript
/srv/salt$ sudoedit pyscript/init.sls
```

init.sls
```yaml
/usr/local/bin/pyhello:
  file.managed:
    - source: salt://pyscript/pyhello
    - mode: '0755'
```

Applied:
```bash
/srv/salt$ sudo salt minion1 state.apply pyscript
minion1:
----------
          ID: /usr/local/bin/pyhello
    Function: file.managed
      Result: True
     Comment: File /usr/local/bin/pyhello updated
     Started: 19:38:41.318564
    Duration: 68.76 ms
     Changes:   
              ----------
              diff:
                  New file
              mode:
                  0755

Summary for minion1
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  68.760 ms
```
And there it is, safe and sound! Profit!   
To verify everything, i went in via SSH, and checked:   
```bash
~/Projects/vagranttest$ vagrant ssh minion1
vagrant@minion1:~$ pyhello
Hello World
vagrant@minion1:~$ pyhello $USER
Hello vagrant
vagrant@minion1:~$ ls -la /usr/local/bin/pyhello 
-rwxr-xr-x 1 root root 115 Nov 18 19:38 /usr/local/bin/pyhello
```
**Working, and everything Ok!**

### d) Laiskaa skriptailua. Tee kansio, josta jokainen skripti kopioituu orjille.
Now that i made some scripts, i could put them all to a single folder, and send it to my minion, but what would be the fun of that?   
I made a script, that makes folder full of scripts!!!! Thats more awesome!   
Lets get it cookin!  
```bash
~$ micro scriptmaker
```

scriptmaker:
```bash
#!/bin/bash
mkdir -p All_Scripts
for ((i=1;i<=$1;i++))
do
	echo -e '#!/bin/bash\necho "Hello World"' > All_Scripts/Hello_World_$i
done
chmod 755 All_Scripts/*
```
Script is ran with `./scriptmaker <number of scripts wanted>`   
It creates a folder `All_Scripts` to `$PWD` and if exists, it overwrites it.
```bash
~$ ./scriptmaker 100
~$ ls All_Scripts/
Hello_World_1    Hello_World_27  Hello_World_45  Hello_World_63  Hello_World_81
Hello_World_10   Hello_World_28  Hello_World_46  Hello_World_64  Hello_World_82
Hello_World_100  Hello_World_29  Hello_World_47  Hello_World_65  Hello_World_83
Hello_World_11   Hello_World_3   Hello_World_48  Hello_World_66  Hello_World_84
Hello_World_12   Hello_World_30  Hello_World_49  Hello_World_67  Hello_World_85
Hello_World_13   Hello_World_31  Hello_World_5   Hello_World_68  Hello_World_86
Hello_World_14   Hello_World_32  Hello_World_50  Hello_World_69  Hello_World_87
Hello_World_15   Hello_World_33  Hello_World_51  Hello_World_7   Hello_World_88
Hello_World_16   Hello_World_34  Hello_World_52  Hello_World_70  Hello_World_89
Hello_World_17   Hello_World_35  Hello_World_53  Hello_World_71  Hello_World_9
Hello_World_18   Hello_World_36  Hello_World_54  Hello_World_72  Hello_World_90
Hello_World_19   Hello_World_37  Hello_World_55  Hello_World_73  Hello_World_91
Hello_World_2    Hello_World_38  Hello_World_56  Hello_World_74  Hello_World_92
Hello_World_20   Hello_World_39  Hello_World_57  Hello_World_75  Hello_World_93
Hello_World_21   Hello_World_4   Hello_World_58  Hello_World_76  Hello_World_94
Hello_World_22   Hello_World_40  Hello_World_59  Hello_World_77  Hello_World_95
Hello_World_23   Hello_World_41  Hello_World_6   Hello_World_78  Hello_World_96
Hello_World_24   Hello_World_42  Hello_World_60  Hello_World_79  Hello_World_97
Hello_World_25   Hello_World_43  Hello_World_61  Hello_World_8   Hello_World_98
Hello_World_26   Hello_World_44  Hello_World_62  Hello_World_80  Hello_World_99
```
Muaaahahaaa!! Working!. Just to verify i checked permissions and testruns:   
```bash
~$ ls -la All_Scripts/Hello_World_1
-rwxr-xr-x 1 sicki sicki 31 18.11. 22:30 All_Scripts/Hello_World_1
~$ ./All_Scripts/Hello_World_1
Hello World
```
To run them all, i made a simple oneliner, but the output would be little too long to paste here:
```bash
for script in $(ls All_Scripts/); do ./All_Scripts/$script; done
```

Now i knew the drill: new state folder, init.sls, script copies and off to the minion!   
```bash
/srv/salt$ sudo mkdir All_Scripts
/srv/salt$ sudoedit All_Scripts/init.sls
```

init.sls:
```bash
/usr/local/bin:
  file.recurse:
    - source: salt://All_Scripts/All_Scripts/
    - file_mode: '0755'
```

Now i copied the script folder to my new state folder:
```bash
/srv/salt$ sudo cp -r ~/All_Scripts All_Scripts/All_Scripts
```

Testrun:   
```bash
/srv/salt$ sudo salt minion1 state.apply All_Scripts
# Shortened, as it didnt even fit the screen
              /usr/local/bin/Hello_World_98:
                  ----------
                  diff:
                      New file
                  mode:
                      0755
              /usr/local/bin/Hello_World_99:
                  ----------
                  diff:
                      New file
                  mode:
                      0755

Summary for minion1
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:   2.141 s
```

Output print is little long... but to verify:
```bash
/srv/salt$ sudo salt minion1 cmd.run 'ls -la /usr/local/bin'
minion1:
    total 420
    drwxr-xr-x  2 root root 4096 Nov 18 20:41 .
    drwxr-xr-x 10 root root 4096 Sep 12 05:16 ..
    -rwxr-xr-x  1 root root   31 Nov 18 20:41 Hello_World_1
    -rwxr-xr-x  1 root root   31 Nov 18 20:41 Hello_World_10
    -rwxr-xr-x  1 root root   31 Nov 18 20:41 Hello_World_100
   .....
   .....
   .....
   .....
    -rwxr-xr-x  1 root root   31 Nov 18 20:41 Hello_World_99
```

In my Minion:
```bash
vagrant@minion1:~$ Hello_World_99
Hello World
```
**All good!**

### e) Intel. Etsi kolme loppuprojektia joltain vanhalta kurssitoteutukselta. Kuvaile projektit tiiviisti, viittaa ja linkitä alkuperäiseeen raporttin. Tässä alakohdassa ei tarvitse vielä kokeilla mitään koneella, vaan voit kuvailla niitä oheismateriaalin perusteella.

**First one up, is my personal favorite, from my friend Niko Heiskanen:**
[Disposable Proxies](https://heiskanen.rocks/server_management/h7)
- Kind of like DIY VPN. 
- Uses `salt-cloud` to rent servers from [Linode](https://www.linode.com/) , to be used as proxy servers, that you can use to route your traffic through. 
- When configured, proxy servers can be made and destroyed with only few commands.
- Contains all the files and instructions for everything necessary.
- A Linode account and some $ is needed.

**Second that caught my attention, was from Hellu:**
[Raspberry Pi as a Bluetooth scanner](https://hellu.home.blog/2020/12/15/7-oma-moduuli-bluetooth-skanneri-saltilla/):
- Sets up Raspberry Pi Minion to work as a Bluetooth scanner
- Nice user friendly setup, where you can get the readings from your Raspberry with single Salt command.
- Uses scriptings and package installing.

**Last but not least, from Otto Hänninen:**
[LAMP, ssh and ufw, with Salt](https://ottohanninen.wordpress.com/2021/05/19/configuration-management-systems-palvelinten-hallinta-spring-2021-h7-oma-moduli/)
- Automatically installs and configures: 
	- apache2
		- Replaces default page with "Hello World"
		- Enables userdir
	- MariaDB
	- php
		- Adds an extension for MariaDB
		-  Sets up apache2 with php
	- ssh
	- ufw
		- opens ports: 22 and 80
- Each are as their own modules and applied with a top.sls

### e) Lukua, ei luottamusta. Kokeile yhtä kohdassa d-Intel löytämääsi modulia koneella. Tämä on _infraa koodina_, joten luottamusta ei tarvita. Voit lukea koodista, mitä olet ajamassa.

As a start, i read about `saltenv` , and more about `fileserver backends`  from:   
[SaltStack: Requesting Files From Specific Environments](https://docs.saltproject.io/en/latest/ref/file_server/environments.html)   
[SaltSatack: Fileserver Modules](https://docs.saltproject.io/en/latest/ref/file_server/all/index.html)    
I figured, i want to have a place to store "test states"   
I configured my Master:   
```bash
~/Projects/vagranttest$ sudoedit /etc/salt/master
```
And added a dev environment to my home folder:
```bash
file_roots:
  base:
    - /srv/salt
  dev:
    - /home/sicki/salt-dev
```
Just a quick recap:    
My masters config looks like this, as there is the `gitfs` fileserver backend added in my previous [h3 - Using salt states from my Github](https://github.com/therealhalonen/configuration_management_systems/blob/master/h3/report.md#using-salt-states-from-my-github-repository).   
And adding to that, i needed to also add `roots` so now it checks Github first, and then local base, when applying states.
```bash
# /etc/salt/master
fileserver_backend:
  - gitfs
  - roots
gitfs_remotes:
  - https://github.com/therealhalonen/salt_states.git

file_roots:
  base:
    - /srv/salt
  dev:
    - /home/sicki/salt-dev
```

Now a quick test run (from home):   
```bash
~$ mkdir salt-dev
~$ cd salt-dev
~/salt-dev$ mkdir development
~/salt-dev$ micro development/init.sls
```
init.sls:
```bash
"echo 'Hello World'":
  cmd.run
```
And testrun, NOTE: `saltenv=dev`  , if not defined, it uses `base`:
```bash
~/salt-dev$ sudo salt minion1 state.apply development saltenv=dev
minion1:
----------
          ID: echo 'Hello World'
    Function: cmd.run
      Result: True
     Comment: Command "echo 'Hello World'" run
     Started: 11:47:15.709241
    Duration: 7.051 ms
     Changes:   
              ----------
              pid:
                  1178
              retcode:
                  0
              stderr:
              stdout:
                  Hello World

Summary for minion1
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:   7.051 ms
```
**UUUUUUUHYEAAH! Working!**
Now that out of the way, i started the actual assignment, after reading again what was it that i needed to do...   

Okay, so i had to install a module made by someone else, so i took:    
[LAMP, ssh and ufw](https://ottohanninen.wordpress.com/2021/05/19/configuration-management-systems-palvelinten-hallinta-spring-2021-h7-oma-moduli/) by Otto Hänninen

Checked his [Github repo for the files](https://github.com/ottohan/LAMP), and all looking right.   
I tested them to my `minion2`:
This time i used my `dev` salt enviroment and cloned the repo straight to my `salt-dev`: 
```bash
~$ cd salt-dev/
~/salt-dev$ git clone https://github.com/ottohan/LAMP .
Cloning into '.'...
remote: Enumerating objects: 49, done.
remote: Counting objects: 100% (49/49), done.
remote: Compressing objects: 100% (41/41), done.
remote: Total 49 (delta 12), reused 20 (delta 1), pack-reused 0
Receiving objects: 100% (49/49), 19.73 KiB | 4.93 MiB/s, done.
Resolving deltas: 100% (12/12), done.
~/salt-dev$ ls
apache  LICENSE  mariadb  php  README.md  ssh  top.sls  ufw
~/salt-dev$ 
```

And as my minion2 is fresh clean install and **safe for testing**, i just applied the `top` straight!   
```bash
~/salt-dev$ sudo salt minion2 state.highstate saltenv=dev
minion2:
----------
          ID: states
    Function: no.None
      Result: False
     Comment: No Top file or master_tops data matches found. Please see master log for details.
     Changes:   

Summary for minion2
------------
Succeeded: 0
Failed:    1
------------
Total states run:     1
Total run time:   0.000 ms
ERROR: Minions returned with non-zero exit code
```
Noooo... It didnt work.   
Bad news is; it doesnt even read the `top.sls`   
Good news is; i like troubleshooting and debugging!!!   
I fired up my `minion3`  and tested as an example, if some state i cloned, works Ok.
```bash
sudo salt minion3 state.apply apache saltenv=dev
```
```bash
Summary for minion3
------------
Succeeded: 5 (changed=5)
Failed:    0
------------
Total states run:     5
Total run time:   9.472 s
```
So that worked! Next i checked the top.sls again.
```bash
~/salt-dev$ cat top.sls
base:
  '*':
    - apache
    - php
    - mariadb
    - ssh
    - ufw
```
***Theres nothing wrong with these files, folders, structure or anything. Its my side!***   
I had an idea! As `yaml` or atleast these .sls files are kind of like folder structures, the `base` caught my eye.    
I quickly checked documentary, before testings: [Salt Project: States, Top](https://docs.saltproject.io/en/latest/ref/states/top.html)   

And so it seems, it might work if: 
- I move everything to my `base` folder `/srv/salt/` 
or
- I move the `top.sls` to my `base` folder `/srv/salt/` 
or
- I edit the `top.sls` and replace `base` with `dev`.    

I decided its easiest to edit the `top.sls`  to look like this:   
```bash
dev:
  '*':
    - apache
    - php
    - mariadb
    - ssh
    - ufw
```
Now i ran it again!
```bash
~/salt-dev$ sudo salt minion2 state.apply saltenv=dev
```
And after a while:   
```bash
#looon loong list of stuff
Summary for minion2
-------------
Succeeded: 15 (changed=14)
Failed:     0
-------------
Total states run:     15
Total run time:   41.700 s
```
15 states totally ran in 41.7s.   
**Working!**   
I ssh:d in to my minion2 and did a few **quick** tests:   

apache2:
```
vagrant@minion2:~$ curl localhost
HelloWorld!
vagrant@minion2:~$ mkdir public_html
vagrant@minion2:~$ echo "Hello Userdir" > public_html/index.html
vagrant@minion2:~$ curl localhost/~vagrant/
Hello Userdir
```
Default site and `userdir` module, all working!

mariadb:
```bash
root@minion2:/home/vagrant# mariadb
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 35
Server version: 10.5.15-MariaDB-0+deb11u1 Debian 11

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SELECT User, Host FROM mysql.user;
+-------------+-----------+
| User        | Host      |
+-------------+-----------+
| mariadb.sys | localhost |
| mysql       | localhost |
| root        | localhost |
+-------------+-----------+
3 rows in set (0.001 sec)

MariaDB [(none)]> 

```
**Working.**

ssh:    
*Is obviously working as i got in, but it was configured already, and the state only checked that its still installed and did nothing.*
```bash
vagrant@minion2:~$ sudo systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2022-11-19 15:55:22 UTC; 41min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 537 (sshd)
      Tasks: 1 (limit: 527)
     Memory: 5.4M
        CPU: 115ms
     CGroup: /system.slice/ssh.service
             └─537 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
```

ufw:
```bash
vagrant@minion2:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
22/tcp (v6)                ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
```
**Working, ports looking like they should.**

For PhP i just checked the version installed:
```bash
vagrant@minion2:~$ php --version
PHP 7.4.33 (cli) (built: Nov  8 2022 11:40:37) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.33, Copyright (c), by Zend Technologies
```
**And its installed.**

**So everything seemed to be working in this module!   
That was fun!**
