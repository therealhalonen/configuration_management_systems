# h3 Git
*Homeworks of third class.    
Handling MarkDown, `git` and: 
Pushing local changes and cloning stuff from Github   
Also voluntary, personal testings with Vagrant, Pillars, Using states from Github and some setups*

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
**[Tero Karvinen himself, in the course lecture](https://terokarvinen.com/2022/palvelinten-hallinta-2022p2) , has been used as a source for the compulsory parts of this homework.**

### a) MarkDown. Tee tämän tehtävän raportti MarkDownina. Helpointa on tehdä raportti GitHub-varastoon, jolloin md-päätteiset tiedostot muotoillaan automaattisesti. Tyhjä rivi tekee kappalejaon, risuaita ‘#’ tekee otsikon, sisennys merkitsee koodinpätkän.

**This is report was written in MarkDown using [Obsidian](https://obsidian.md/).    
See [here](https://raw.githubusercontent.com/therealhalonen/configuration_management_systems/master/h3/report.md) for the raw format.**

---
### b) Offline. Tee paikallinen offline-varasto `git`:llä. Varaston nimessä tulee olla sana "cat" (kissa). Aiemmin tehty varasto ei siis kelpaa. Aseta itsellesi sähköpostiosoite ja nimi. Näytä varastollasi muutosten teko ja niiden katsominen lokista.

Okey dokey.   
Started with making a simple folder, named `catdir` and moving inside it.
```bash
~$ mkdir catdir && cd catdir
``` 
Now initialized git:
```bash
~/catdir$ git init
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint: 
hint: 	git config --global init.defaultBranch <name>
hint: 
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint: 
hint: 	git branch -m <name>
Initialized empty Git repository in /home/sicki/catdir/.git/
```
Git was initialized, and default branch is `master`   
Ive always used `master` as the default, so reading the `hints` first, which i didnt like to read again, i added the `master` as the `default branch` to global git config:   
*(Global config, is the one that gets read, when git checks for some info about me and what ive wanted to configure, and it applies to every git repository i make, if i dont config something specific locally to that certain repo)*
```bash
~/catdir$ git config --global init.defaultBranch master
```
There.    
Then i wanted my credentials; username and email to be also added to global git config:   
Username: Sicki   
Email: my-personal@mail.com
```bash
git config --global user.email mypersonal@mail.com
git config --global user.name "Sicki"
```
To verify they are all there in the config, i checked:
```bash
~/catdir$ git config --global --list
user.name=Sicki
user.email=mypersonal@mail.com
init.defaultbranch=master
```
**Done** 

Now i started to do some stuff to my local git repository.   
README.md:
```bash
~/catdir$ echo -e "# Welcome\nThis my local git repository" > README.md 
~/catdir$ cat README.md
# Welcome
This my local git repository
```
There it was, nice and pretty README file, written in MarkDown also.   
I think that was enough about creating, now i was off to commiting the changes:   
First i check the status (not actually necessary):
```bash
~/catdir$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	README.md

nothing added to commit but untracked files present (use "git add" to track)
```
Okey, so nothing added to commit!   
Then i added everything with `git add .`  and just to show, i checked the status again:
```bash
~/catdir$ git add .
~/catdir$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   README.md
```
Now the README file was added for the commit.    
And i needed to commit it with `git commit`
```bash               
Add README.md
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch master
#
# Initial commit
#
# Changes to be committed:
#       new file:   README.md
#

```
A new view opened, where i typed the commit message `Add README.md`   
Save and Exit.   
And i ended up like this:
```bash
~/catdir$ git commit
[master (root-commit) 9e2b8ba] Add README.md
 1 file changed, 2 insertions(+)
 create mode 100644 README.md
```
Which tells that the commit has been made, 1 file is changed and 2 insertions have been made to my local git repository (created file and write content to file)

Next i made  few commits to fill the log a little bit:
```bash
~/catdir$ echo One > one.file
~/catdir$ git add . && git commit -m "Add a commit" 
[master 847da89] Add a commit
 1 file changed, 1 insertion(+)
 create mode 100644 one.file
~/catdir$ echo Two > Two.file
~/catdir$ git add . && git commit -m "Add a second commit" 
[master 5f19f6e] Add a second commit
 1 file changed, 1 insertion(+)
 create mode 100644 Two.file
```

And then i checked the log with `git log`:
```bash
~/catdir$ git log
commit 5f19f6e9153e61db3b57edeb214a10c6ba1d2bf4 (HEAD -> master)
Author: Sicki <mypersonal@mail.com>
Date:   Fri Nov 11 13:09:50 2022 +0200

    Add a second commit

commit 847da89384ec12dedb39dc2d073ac63cca0bb1ec
Author: Sicki <mypersonal@mail.com>
Date:   Fri Nov 11 13:09:37 2022 +0200

    Add a commit

commit 9e2b8baa7d1503c8ed48f72f45f0b1d665409940
Author: Sicki <mypersonal@mail.com>
Date:   Fri Nov 11 13:03:49 2022 +0200

    Add README.md
```
There was 3 commits in the log. First with commit ID `9e2b8baa7d1503c8ed48f72f45f0b1d665409940` was the one i made first, when i added the README.md

And for more detailed  with `git log --patch`:
```bash
~/catdir$ git log --patch
commit 5f19f6e9153e61db3b57edeb214a10c6ba1d2bf4 (HEAD -> master)
Author: Sicki <mypersonal@mail.com>
Date:   Fri Nov 11 13:09:50 2022 +0200

    Add a second commit

diff --git a/Two.file b/Two.file
new file mode 100644
index 0000000..3b0086f
--- /dev/null
+++ b/Two.file
@@ -0,0 +1 @@
+Two

commit 847da89384ec12dedb39dc2d073ac63cca0bb1ec
Author: Sicki <mypersonal@mail.com>
Date:   Fri Nov 11 13:09:37 2022 +0200

    Add a commit

diff --git a/one.file b/one.file
new file mode 100644
index 0000000..3609f20
--- /dev/null
+++ b/one.file
@@ -0,0 +1 @@
+One

commit 9e2b8baa7d1503c8ed48f72f45f0b1d665409940
Author: Sicki <mypersonal@mail.com>
Date:   Fri Nov 11 13:03:49 2022 +0200

    Add README.md

diff --git a/README.md b/README.md
new file mode 100644
index 0000000..ed55782
--- /dev/null
+++ b/README.md
@@ -0,0 +1,2 @@
+# Welcome
+This my local git repository
```

---
### c) Doh! Tee tyhmä muutos gittiin, älä tee commit:tia. Tuhoa huonot muutokset ‘git reset --hard’. Huomaa, että tässä toiminnossa ei ole peruutusnappia.

**Continuing straight from assignment b):**   
I made some stupid things.... with `rm` ... inside my repository...
```
~/catdir$ rm *
```
Nooo! Now i checked the status:
```bash
~/catdir$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	deleted:    README.md
	deleted:    Two.file
	deleted:    one.file

no changes added to commit (use "git add" and/or "git commit -a")
```
All deleted! But! Not lost! Because Git is awesome.   
To get em back, i used a simple `reset`:
```bash
~/catdir$ git reset --hard
HEAD is now at 5f19f6e Add a second commit
```
And i saw, it has reset to my last commit!   
Then, as always, to verify i checked status, short log to see commit ID:s and current directory content:
```bash
~/catdir$ git status && git log|grep "commit .*"
On branch master
nothing to commit, working tree clean
commit 5f19f6e9153e61db3b57edeb214a10c6ba1d2bf4
commit 847da89384ec12dedb39dc2d073ac63cca0bb1ec
commit 9e2b8baa7d1503c8ed48f72f45f0b1d665409940
~/catdir$ ls -la
total 24
drwxr-xr-x  3 sicki sicki 4096 11.11. 14:04 .
drwxr-xr-x 32 sicki sicki 4096 11.11. 12:54 ..
drwxr-xr-x  8 sicki sicki 4096 11.11. 14:08 .git
-rw-r--r--  1 sicki sicki    4 11.11. 14:04 one.file
-rw-r--r--  1 sicki sicki   39 11.11. 14:04 README.md
-rw-r--r--  1 sicki sicki    4 11.11. 14:04 Two.file

```
Everything is back to normal!   
**Profit!!**

---
### d) Online. Tee uusi varasto GitHubiin (tai Gitlabiin tai mihin vain vastaavaan palveluun). Varaston nimessä ja lyhyessä kuvauksessa tulee olla sana "car" (auto). Aiemmin tehty varasto ei kelpaa. (Muista tehdä varastoon tiedostoja luomisvaiheessa, suosittelen tekemään README.md ja vapaista lisensseistä itse tykkään GPLv3 eli GNU General Public License, version 3)

For this assignment, i used Github.com, as thats what im familiar with.   
*I wont go through the Sign Up here*

As i already had an account there, i started with creating a new repository.   
From my repository view, clicked green "New" tab.   
![Image1](https://github.com/therealhalonen/configuration_management_systems/blob/master/h3/res/repo.png)
That lead to a new place, where i could:
- Give the new repository:
	- A name
	- A description
	- Decide if i want it Public or Private
	- Add a README file automatically
	- Add .gitignore ( with what you can decide if you want certain files to not be pushed from your local git repo to the remote one)
	- Choose a license

I made a simple car_dir repo, like this:   
![Image2](https://github.com/therealhalonen/configuration_management_systems/blob/master/h3/res/new_repo.png)   
Note! Before i clicked `Create repository`, i changed the default branch to `master` which i use as a default locally.   
And it appeared to https://github.com/therealhalonen/car_nissan_connect   
![Image3](https://github.com/therealhalonen/configuration_management_systems/blob/master/h3/res/ready_repo.png)

---
### e) Dolly. Kloonaa edellisessä kohdassa tehty varasto itsellesi, tee muutoksia, puske ne palvelimelle, ja näytä, että ne ilmestyvät weppiliittymään.

I wanted my previously created online repository to my local git.   
First i needed the full ssh address to my Github repository that i wanted to clone.   
It could be obtained by clicking the green "Code" button:   
![Image4](https://github.com/therealhalonen/configuration_management_systems/blob/master/h3/res/get_address.png)   
Note: To save time, i have learned the syntax of the address:   
```bash
git@github.com:<username>/<repositoryname>
``` 

First :
```bash
~$ git clone git@github.com:therealhalonen/car_nissan_connect.git
Cloning into 'car_nissan_connect'...
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (4/4), 12.53 KiB | 4.18 MiB/s, done.
```
Then i went to the folder it created:   
```bash
~$ cd car_nissan_connect/
~/car_nissan_connect$ ls
LICENSE  README.md
~/car_nissan_connect$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```
I saw it has cloned the repo succesfully and contains the files i made during the creation in Github.   
Now i wanted to add stuff:
```bash
~/car_nissan_connect$ mkdir src && echo "Important things" > src/source
~/car_nissan_connect$ git add . && git commit
[master 70a2ef2] Add part of source code
 1 file changed, 1 insertion(+)
 create mode 100644 src/source
```
Now i had added a folder and file locally.    
Next i wanted to push it back to my Github repo.   
First i made SSH-keys, which i actually had already, but made new one to show the process with `ssh-keygen`:
```bash
~/car_nissan_connect$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/sicki/.ssh/id_rsa): /home/sicki/.ssh/id_github_test
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/sicki/.ssh/id_github_test
Your public key has been saved in /home/sicki/.ssh/id_github_test.pub
``` 
In my ~/.ssh i searched the created **PUBLIC** key:
```bash
~/car_nissan_connect$ ls ../.ssh|grep id_git*..*.pub
id_github_test.pub
```
And as it were there, like supposed to. I took the content of the file, and copied it to my https://github.com/settings/keys -> New SSH key.   
![Image5](https://github.com/therealhalonen/configuration_management_systems/blob/master/h3/res/add_key.png)   
Add SSH key to confirm.   

Then i tested that the ssh connection to github is working for me.
```bash
~/car_nissan_connect$ ssh -T git@github.com
Hi therealhalonen! You've successfully authenticated, but GitHub does not provide shell access.
```
**Working!**

After that, i could just push my changes to my remote repository (Github).
```bash
~/car_nissan_connect$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 12 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (4/4), 372 bytes | 372.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:therealhalonen/car_nissan_connect.git
   48266ec..c86624b  master -> master
```
To verify, i went to https://github.com/therealhalonen/car_nissan_connect/commits/master to see that it has the locally made commit there!   
![Image6](https://github.com/therealhalonen/configuration_management_systems/blob/master/h3/res/commit_success.png)

---
**As the next two assignments were voluntary;**
```
f) Vapaaehtoinen: Tee uusi salt-moduli. Voit asentaa ja konfiguroida minkä vain uuden ohjelman: demonin, työpöytäohjelman tai komentokehotteesta toimivan ohjelman. Käytä tarvittaessa ‘find -printf “%T+ %p\n”|sort’ löytääksesi uudet asetustiedostot. (Tietysti eri ohjelma kuin aiemmissa tehtävissä, tarkoitushan on harjoitella Salttia)

g) Vapaaehtoinen: Laita salt gittiin. Tee uusi moduli. Kloonaa varastosi toiselle koneelle (tai poista srv/salt ja palauta se kloonaamalla) ja jatka sillä. (Salt tiedostot mistä vain hakemistosta, huomaa suhteellinen polku: 'sudo salt-call --local --file-root srv/salt/ state.apply')
```
**, i skipped them but did something similar.**

## `echo $RANDOM`)
 - Learn how Pillars work.
 - Create some salt states.
 - Learn what is Vagrant and how to use it with Salt.
 - Use salt states from Github repository.

---
### Using Pillars
*Pillars are tree-like structures of data defined on the Salt Master and passed through to minions. They allow confidential, targeted data to be securely sent only to the relevant minion. - [SaltProject: Pillar](https://docs.saltproject.io/en/latest/topics/tutorials/pillar.html)*

First i learned about what Pillars are and how to handle secret stuff in `Salt`.   
Sources used:
- [Saltproject Docs - Pillar](https://docs.saltproject.io/en/latest/topics/tutorials/pillar.html)
- [Linode.com - Secrets management with salt](https://www.linode.com/docs/guides/secrets-management-with-salt/)
- [Tero Karvinen - Simple secrets in salt pillars](https://terokarvinen.com/2018/simple-secrets-in-salt-pillars/)
- [Niko Heiskanen himself](https://heiskanen.rocks/)

I created a folder `/srv/pillar`   
Inside i made `top.sls`  that contains:   
```python
base:
  '*':
    - secret_stuff
```

And `secret_stuff.sls` that contains:   
All the hashed passwords, needed for states.   
Sample file:
```python
#secre_stuff.sls
superior_pass: <password_hash>
regular_pass: <password_hash>
```

Usage in state file:
```python
do_something:
  - password: {{ pillar['regular_pass'] }}
```
[Sample state file with Pillar usage]()

---
### Creating salt states

In my previous reports i created some salt states, and used thise as a source to create more.   
Also read alot about them from:
- [SaltProject - Fundamentals - States](https://docs.saltproject.io/en/getstarted/fundamentals/states.html)
- [Tero Karvinen - Salt states](https://terokarvinen.com/2018/salt-states-i-want-my-computers-like-this/)
And of course  **[Tero Karvinen himself, in the course lecture](https://terokarvinen.com/2022/palvelinten-hallinta-2022p2)**

What i did can be found in [my Github Repo](https://github.com/therealhalonen/salt_states)   
*Some of them requires Pillars, as i used Pillars to keep hashed passwords in user handlings.*   
I store them in my local machine.

---
### Vagrant with Salt
*Vagrant is designed for everyone as the easiest and fastest way to create a virtualized environment! - [Vagrant - Intro](https://developer.hashicorp.com/vagrant/intro)*   

First i installed `vagrant`:    
Getting the latest version of Vagrant for Debin:   
Source: https://developer.hashicorp.com/vagrant/downloads
```bash
 wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
 echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```
```bash
~$ sudo apt-get update && sudo apt-get install vagrant
```

After Vagrant was installed succesfully, i "made" a script to create VM minions, and by "made", i mean i used the following sources to get it like i wanted:
- [Tero Karvinen - 2 machine Vagrant setup](https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/)   
- [Tero Karvinen - Install and boot a new VM in 31sec](https://terokarvinen.com/2017/04/11/vagrant-revisited-install-boot-new-virtual-machine-in-31-seconds/)   
- [Tero Karvinen - Automatically provision Vagrant VMs as Salt slaves](https://terokarvinen.com/2018/automatically-provision-vagrant-virtualmachines-as-salt-slaves/)
- [Superuser - Run script on host during Vagrant up](https://superuser.com/questions/701735/run-script-on-host-machine-during-vagrant-up)
- [Niko Heiskanen - Himself and his page](https://heiskanen.rocks/server_management/h2)

***If you think about using some custom stuff when creating VM:s with Vagrant, i suggest you read the articles above first.***

Final version of the script:
```bash
#Vagrantfile
$common = <<COMMON
apt update -y
sudo apt install bash-completion -y
COMMON

$minion = <<MINION
apt install salt-minion -qy
sed -i 's/\#master\:.*/master\:\ <IP>/g' /etc/salt/minion
systemctl restart salt-minion
MINION

$minion_count = <MINIONS_NEEDED>

Vagrant.configure("2") do |config|
        config.vm.provision "add thing", type: "shell", inline: $common
        config.vm.box = "debian/bullseye64"

        (1..$minion_count).each do |i|
                config.vm.define "minion#{i}" do |slave|
                        slave.vm.provision "final", type: "shell", inline: $minion
                        slave.vm.network "private_network", type: "dhcp"
                        slave.vm.hostname = "minion#{i}"
                        slave.vm.provider "virtualbox" do |v|
                                v.memory = 512
                                v.cpus = 1
                                v.name = "Vagrant-Minion#{i}"
                        end
                end
        end
end
```
To break it down:
- It creates <MINIONS_NEEDED> amount of Debian 11, VirtualBox VM:s to you.
- It "injects" Masters address to every Minions /etc/salt/minion, so they are automatically connecting to Master, the correct IP needs to be edited to script first to replace the <'IP'>

To use the file, i made a folder and inside the folder i made a `Vagrantfile` 
```
~$ mkdir vagranttest
~$ cd vagranttest 
~$ touch Vagrantfile
``` 
To Vagrantfile i pasted the script content.   
I changed the IP to my machines IP and put `4` for Minion count.   
Now i started my VirtualBox, then Vagrant with `vagrant up`
```bash
~/Projects/vagranttest$ vagrant up
Bringing machine 'minion1' up with 'virtualbox' provider...
Bringing machine 'minion2' up with 'virtualbox' provider...
Bringing machine 'minion3' up with 'virtualbox' provider...
Bringing machine 'minion4' up with 'virtualbox' provider...
...
... Lots of stuff here about fetching the VM:s from https://app.vagrantup.com/boxes
... Lots of stuff here about making the VM:s and updating them
... More lots of stuff about something something
... 
``` 

There was a lot stuff going on after `vagrant up` but to verify the machines, i tested `vagrant status`

```bash
~/vagranttest$ vagrant status
Current machine states:

minion1                   running (virtualbox)
minion2                   running (virtualbox)
minion3                   running (virtualbox)
minion4                   running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
~/vagranttest$ 
```
All up and running.   

I checked keys and accepted them:
```bash
~/vagranttest$ sudo salt-key
Accepted Keys:
Denied Keys:
Unaccepted Keys:
minion1
minion2
minion3
minion4
Rejected Keys:
~/vagranttest$ sudo salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
minion1
minion2
minion3
minion4
Proceed? [n/Y] y
Key for minion minion1 accepted.
Key for minion minion2 accepted.
Key for minion minion3 accepted.
Key for minion minion4 accepted.

```

And lastly, tested the connection.
```bash
~/vagranttest$ sudo salt '*' test.ping
minion4:
    True
minion1:
    True
minion3:
    True
minion2:
    True
```

Everyone answered my `ping`.   
All working as they should! Now i have 4 Minions ready to obey.   
To shut them down:
- `vagrant halt`
To destroy them, and files related to them
- `vagrant destroy`
And `vagrant up` to either bring them up or creating them!   
Peace!

---
### Using salt states from my Github repository

Took all my states from my local `/srv/salt/` to my `~/salt_states` and changed the owner and permissions.
```bash
mkdir ~/salt_states
sudo cp -r /srv/salt/* ~/salt_states/
sudo chmod -R 0750 salt_states
sudo chown sicki:sicki salt_states
cd ~/salt_sates
```

This point i already had a Github repository for my salt states ready in [my Github repository](https://github.com/therealhalonen/salt_states) and ready to be pushed there.    
I did it the "hard way", but to avoid that you should do it like this:
- Create a repository `salt_states` to your Github repositories.
- Clone it to your machine
- Make sure local and remote are up to date.
- After that copy the to stuff like i did above, and check that permissions are okay.
- git add . && git commit && git push , and they are synced on local and remote.

**Now that i had the states in my Github repo, i read reliable sources:**
- [SaltStacks - Tutorials - gitfs](https://docs.saltproject.io/en/3003/topics/tutorials/gitfs.html)
- [Roylarsen - Saltstack gitfs](https://www.roylarsen.xyz/saltstack-gitfs/)
- [Niko Heiskanen - Himself and his site](https://heiskanen.rocks/server_management/h3)

As working in my Master machine, i went to edit `/etc/salt/master` config.
```bash
~/$ sudoedit /etc/salt/master
```

The file contained everything commented, so i just cleared it and put this in:
```bash
#master
fileserver_backend:
  - gitfs
gitfs_remotes:
  - https://github.com/therealhalonen/salt_states.git
```

Then i removed my salt states from my local storage and restarted the master daemon:
```
sudo rm -rf /srv/salt/*
sudo systemctl restart salt-master
```

To verify the states work, i used the setup i did in the [Vagran part](https://github.com/therealhalonen/configuration_management_systems/blob/master/h3/report.md#vagrant-with-salt), and proceeded with applying a state.
```bash
~/vagranttest$ sudo salt '*' state.apply apache2
```

Everything was working and state from [here](https://github.com/therealhalonen/salt_states/blob/master/apache2/init.sls)  , got applied to every Minion (4)   
The print was 1km long, so i didnt paste it here...   
But to verify i went through each Minion with ssh:   
```bash
~/vagranttest$ vagrant ssh minion1
Linux minion1 5.10.0-18-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Nov 11 16:37:35 2022 from 10.0.2.2
vagrant@minion1:~$ curl localhost
Hello World!
This is your front page
```
**And yes, it was working!!**
