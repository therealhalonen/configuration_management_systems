# Testing my project.

This is the report of testing my Saltstack project in the course:   
[Configuration Management Systems - Palvelinten Hallinta by Tero Karvinen](https://terokarvinen.com/2022/palvelinten-hallinta-2022p2/)
Report of the making: [HERE](https://github.com/therealhalonen/domain_control)
Project itself: [HERE]

When the project was ready, after last fine tunings,  i made some tests to see if its really functional as it should.   

I started reporting this, from a point where i have all Minions up and connected to Master:
```bash
~/Projects/project_folder$ sudo salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
dev-fedora-ws
dev-server1
dev-server2
dev-ubuntu
fileserver
ubuntu-ws
webserver
windows-ws
Proceed? [n/Y] y
Key for minion dev-fedora-ws accepted.
Key for minion dev-server1 accepted.
Key for minion dev-server2 accepted.
Key for minion dev-ubuntu accepted.
Key for minion fileserver accepted.
Key for minion ubuntu-ws accepted.
Key for minion webserver accepted.
Key for minion windows-ws accepted.
```
As a testrun, i ran my `hello_all` state: 
```bash
~/Projects/project_folder$ sudo salt '*' state.apply hello_all
fileserver:
----------
          ID: echo Hello $(hostname)!
    Function: cmd.run
      Result: True
     Comment: Command "echo Hello $(hostname)!" run
     Started: 18:57:39.240516
    Duration: 11.809 ms
     Changes:   
              ----------
              pid:
                  2512
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
Total run time:  11.809 ms
webserver:
----------
          ID: echo Hello $(hostname)!
    Function: cmd.run
      Result: True
     Comment: Command "echo Hello $(hostname)!" run
     Started: 18:57:39.241007
    Duration: 14.756 ms
     Changes:   
              ----------
              pid:
                  2511
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
Total run time:  14.756 ms
ubuntu-ws:
----------
          ID: echo Hello $(hostname)!
    Function: cmd.run
      Result: True
     Comment: Command "echo Hello $(hostname)!" run
     Started: 18:57:38.424803
    Duration: 7.475 ms
     Changes:   
              ----------
              pid:
                  4232
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
Total run time:   7.475 ms
dev-server1:
----------
          ID: echo Hello DEV! $(hostname) DEV!
    Function: cmd.run
      Result: True
     Comment: Command "echo Hello DEV! $(hostname) DEV!" run
     Started: 18:57:39.666310
    Duration: 11.858 ms
     Changes:   
              ----------
              pid:
                  2491
              retcode:
                  0
              stderr:
              stdout:
                  Hello DEV! dev-server1 DEV!

Summary for dev-server1
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  11.858 ms
dev-server2:
----------
          ID: echo Hello DEV! $(hostname) DEV!
    Function: cmd.run
      Result: True
     Comment: Command "echo Hello DEV! $(hostname) DEV!" run
     Started: 18:57:39.689760
    Duration: 17.664 ms
     Changes:   
              ----------
              pid:
                  2506
              retcode:
                  0
              stderr:
              stdout:
                  Hello DEV! dev-server2 DEV!

Summary for dev-server2
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  17.664 ms
dev-ubuntu:
----------
          ID: echo Hello DEV! $(hostname) DEV!
    Function: cmd.run
      Result: True
     Comment: Command "echo Hello DEV! $(hostname) DEV!" run
     Started: 18:57:39.824139
    Duration: 6.962 ms
     Changes:   
              ----------
              pid:
                  2970
              retcode:
                  0
              stderr:
              stdout:
                  Hello DEV! dev-ubuntu DEV!

Summary for dev-ubuntu
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:   6.962 ms
dev-fedora-ws:
----------
          ID: echo Hello DEV! $(hostname) DEV!
    Function: cmd.run
      Result: True
     Comment: Command "echo Hello DEV! $(hostname) DEV!" run
     Started: 13:57:40.149067
    Duration: 7.868 ms
     Changes:   
              ----------
              pid:
                  6635
              retcode:
                  0
              stderr:
              stdout:
                  Hello DEV! dev-fedora-ws DEV!

Summary for dev-fedora-ws
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:   7.868 ms
windows-ws:
----------
          ID: echo Hello %COMPUTERNAME%!
    Function: cmd.run
      Result: True
     Comment: Command "echo Hello %COMPUTERNAME%!" run
     Started: 10:57:41.741129
    Duration: 126.873 ms
     Changes:   
              ----------
              pid:
                  4836
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
Total run time: 126.873 ms
```
**Working!**   
If you read my report before this testing, you now probably see and understand, how it applied kind of like the same state, but from different location, depending on the enviroment!    
COOOL!   

Another quick test, is to apply everrrrrything, but i will leave the output short:
```bash
~$ sudo salt '*' state.apply | grep Failed -B 3
Summary for dev-ubuntu
------------
Succeeded: 1 (changed=1)
Failed:    0
--
Summary for dev-server2
------------
Succeeded: 1 (changed=1)
Failed:    0
--
Summary for dev-server1
------------
Succeeded: 1 (changed=1)
Failed:    0
--
Summary for dev-fedora-ws
------------
Succeeded: 1 (changed=1)
Failed:    0
--
Summary for webserver
-------------
Succeeded: 13 (changed=12)
Failed:     0
--
Summary for fileserver
-------------
Succeeded: 14 (changed=12)
Failed:     0
--
Summary for windows-ws
------------
Succeeded: 3 (changed=3)
Failed:    0
--
Summary for ubuntu-ws
------------
Succeeded: 6 (changed=5)
Failed:    0
```
**All seemed to be working!**

After that to do little more testing:    
Logged in to my `ubuntu-ws` Minion:   
Tested SSH, Avahi and Apache2:    
![Image1](https://github.com/therealhalonen/configuration_management_systems/blob/master/h7/res/ssh_apache_avahi.png)     
Regular user `sicki` didnt get in, but `supreme` got in, and had `sudo` rights as should!    
Also Avahi is running because i could use `.local`  and apache2 seems rolling!   
Excellent!   

Now to test browser, it should be Firefox-ESR and should show the apache 2 page when browsing to `webserver.local`    

![Image2](https://github.com/therealhalonen/configuration_management_systems/blob/master/h7/res/firefox.png)


![Image3](https://github.com/therealhalonen/configuration_management_systems/blob/master/h7/res/apache_browser.png)   
There they are! All working.   

Final test i do for now, Samba:       
Saw the Fileserver already available in `other locations` in filebrowser, and logged in as `sambauser` which was defined and created in the state-file. 

![Image4](https://github.com/therealhalonen/configuration_management_systems/blob/master/h7/res/samba.png)    
Created a folder succesfully!! Excellent!

**I will leave testing here, and it will be more thorough in the live demonstration during the final class of the course!**

