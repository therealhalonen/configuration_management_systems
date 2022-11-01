# h1 Hello Salt
*Homeworks of first class. 
Superquick Linux commandline recap, getting to know what is `Salt` and why, how to apply states in `salt` and how to gather info with `grains` .* 

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
**x) Lue ja tiivistä (Tässä x-alakohdassa pelkkä luku / katselu / kuuntelu riittää, ei tarvitse tehdä testejä tietokoneella. Muutama ranskalainen viiva per artikkeli riittää)**

- Karvinen 2020: [Command Line Basics Revisited](http://terokarvinen.com/2020/command-line-basics-revisited/)

  - *Notes from the article:*
    
    - Useful very basic Linux commands and important directories to remember:

```python
# Browsing system and files, making changes:
pwd | ls | cd <directory> | less | mkdir | mv | cp (-r) | rmdir <directory | rm | nano
# ssh remote stuff:
ssh username@<remote> | scp <folder_from> username@<remote>:folder_to/ 
# To get help:
man <something> | <something> --help | <something> -h |
# Keep system up to date:
sudo apt-get <update> <upgrade> <dist-upgrade>
# Install and remove stuff
sudo apt-get install <package_name> | sudo apt-get purge <package_name> 
# Tips for speed  
[tab] | [tab][tab] | history | CTRL+r
```

```python
/ # Root directory, the top of the filesystem.
/home/ # Contains home directories for all users.
/home/$USER/ # Home directory of current user. The only place where user can permanently store data.
/etc/ # All system wide settings.
/media/ # Removable media.
/var/log/ # System wide logs
```

- Karvinen 2018: [Salt States – I Want My Computers Like This](http://terokarvinen.com/2018/salt-states-i-want-my-computers-like-this)

  - *As ive only used salt in `--local` i gathered some notes from the article:* 
```python
# Masters folder that holds all the instructions as .sls, to minions are in (have to be made first):
/srv/salt/
# .sls = states = instructions to minions are in format:
/tmp/filename.txt:
  file.managed:
    - source: salt://filename.txt
# filename.txt could contain for example:
Hello World
# To apply a single state once on all minions
sudo salt '*' state.apply <statename> # = .sls filename

# For automation. top.sls, in /srv/salt/
base:
  '*':
    - hello
# Apply immediately:
sudo salt '*' state.highstate
```
- Karvinen 2006: [Raportin kirjoittaminen](https://terokarvinen.com/2006/raportin-kirjoittaminen-4/?fromSearch=raportin%20kirjoittaminen)

  - *Notes to writing an awesome report:*

    - **Make it repeatable.** 
      - Same enviroment, must have same outcome, no matter who does it. Give proper info of the used enviroment.

    - **Be precise.**
      - Commands used. Timings if relevant. Did it work? If not, why? Should it? How it failed if it did? Is it because of the tools used?  PC not working? 

    - **Write in the imperfect.**

    - **Make it easy to read.**
      - Format correctly, use headings and sub-headings. Check and fix typos.

    - **Always remember to cite source references!**

---


**Articles above and [Tero Karvinen himself, in the course lecture](https://terokarvinen.com/2022/palvelinten-hallinta-2022p2) , has been used as a source for this homework.**

**a) Tee Saltilla tiedosto yksittäisellä komentorivillä (state.single)**

Started the assigment by installing `salt-master` and `salt-minion`
```bash
sudo apt-get install salt-master salt-minion
```

Created an empty file named `homework_test` to `/tmp/`
```bash
~$ sudo salt-call --local state.single file.managed /tmp/homework_test
[WARNING ] State for file: /tmp/homework_test - Neither 'source' nor 'contents' nor 'contents_pillar' nor 'contents_grains' was defined, yet 'replace' was set to 'True'. As there is no source to replace the file with, 'replace' has been set to 'False' to avoid reading the file unnecessarily.
local:
----------
          ID: /tmp/homework_test
    Function: file.managed
      Result: True
     Comment: Empty file
     Started: 15:11:34.747742
    Duration: 5.445 ms
     Changes:   
              ----------
              new:
                  file /tmp/homework_test created

Summary for local
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
```

---

**b) Tee saltille idempotentti, infra koodina hei maailma (siis tiedostosta, /srv/salt/foo.sls)**

I made the path to salt, and made/edited a foo.sls config file:
```bash
~$ sudo mkdir -p /srv/salt/
~$ sudo nano /srv/salt/foo.sls
```

```bash
#foo.sls
/tmp/hello_world:
  file.managed:
    - contents: "Hello World"
```

Then applied the content:
```bash
~$ sudo salt-call --local state.apply foo
local:
----------
          ID: /tmp/hello_world
    Function: file.managed
      Result: True
     Comment: File /tmp/hello_world updated
     Started: 15:32:08.397691
    Duration: 7.444 ms
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
Total run time:   7.444 ms
```
It created a new file correctly, so i checked that the content is as it should:
```bash
~$ cat /tmp/hello_world
Hello World
```
Niiiice! All good.

---

**d) Kerää tietoa koneesta saltin avulla (grains.items)**

While `sudo salt-call --local grains.items` returns so much stuff, i tested the `grains.item` command with parameters for specific stuff i wanted to see:
```bash
sudo salt-call --local grains.item localhost lsb_distrib_description kernel kernelversion shell
local:
    ----------
    kernel:
        Linux
    kernelversion:
        #1 SMP Debian 5.10.140-1 (2022-09-02)
    localhost:
        vb-debian
    lsb_distrib_description:
        Debian GNU/Linux 11 (bullseye)
    shell:
        /bin/bash
```

---

**e) Kokeile jotain toista tilaa kuin file.managed. Tärkeitä ovat pkg.installed, file.managed, service.running, file.symlink, user.present, group.present. Ohjeita saa esim "sudo salt-call --local sys.state_doc pkg.installed|less"restart**

For this assignment i tested `pkg.installed`:
```bash
sudo salt-call --local state.single pkg.installed apache2
local:
----------
          ID: apache2
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 16:12:24.290378
    Duration: 49.652 ms
     Changes:   

Summary for local
------------
Succeeded: 1
Failed:    0
------------
Total states run:     1
Total run time:  49.652 ms
```
`service.running`
```bash
~$ sudo salt-call --local state.single service.running avahi-daemon
local:
----------
          ID: avahi-daemon
    Function: service.running
      Result: True
     Comment: The service avahi-daemon is already running
     Started: 16:18:21.660081
    Duration: 25.932 ms
     Changes:   

Summary for local
------------
Succeeded: 1
Failed:    0
------------
Total states run:     1
Total run time:  25.932 ms
```
`user.present` : 
```bash
~$ sudo salt-call --local state.single user.present test
local:
----------
          ID: test
    Function: user.present
      Result: True
     Comment: New user test created
     Started: 16:19:50.702742
    Duration: 50.704 ms
     Changes:   
              ----------
              fullname:
              gid:
                  1001
              groups:
                  - test
              home:
                  /home/test
              homephone:
              name:
                  test
              other:
              passwd:
                  x
              roomnumber:
              shell:
                  /bin/sh
              uid:
                  1001
              workphone:

Summary for local
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  50.704 ms
```
To remove that user `test` , i used `user.absent` :
```bash
~$ sudo salt-call --local state.single user.absent test
local:
----------
          ID: test
    Function: user.absent
      Result: True
     Comment: Removed user test
     Started: 16:20:46.906152
    Duration: 61.673 ms
     Changes:   
              ----------
              test:
                  removed
              test group:
                  removed

Summary for local
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  61.673 ms
```
`group.present`
```bash
~$ sudo salt-call --local state.single group.present testgroup
local:
----------
          ID: testgroup
    Function: group.present
      Result: True
     Comment: New group testgroup created
     Started: 16:21:29.173004
    Duration: 29.418 ms
     Changes:   
              ----------
              gid:
                  1001
              members:
              name:
                  testgroup
              passwd:
                  x

Summary for local
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  29.418 ms
```

And to remove that group `testgroup` , i used `group.absent`:
```bash
~$ sudo salt-call --local state.single group.absent testgroup
local:
----------
          ID: testgroup
    Function: group.absent
      Result: True
     Comment: Removed group testgroup
     Started: 16:21:54.046750
    Duration: 27.36 ms
     Changes:   
              ----------
              testgroup:

Summary for local
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  27.360 ms
```
