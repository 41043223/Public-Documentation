## Block ssh attack

A hacker attack my server between July 5 and July 9(I have been blocked it manually)
He attack 21813 times from ```164.92.200.62``` and use root to log in first, but root have been limit in LAN, so he uses dictionary attack try to hack my server

```sh
Jul  5 07:14:52 ccis-server-5 sshd[707256]: Failed password for invalid user root from 164.92.200.62 port 39868 ssh2
Jul  5 07:15:03 ccis-server-5 sshd[707272]: Failed password for invalid user root from 164.92.200.62 port 41772 ssh2
Jul  5 07:15:13 ccis-server-5 sshd[707284]: Failed password for invalid user root from 164.92.200.62 port 43564 ssh2
Jul  5 07:15:23 ccis-server-5 sshd[707294]: Failed password for invalid user root from 164.92.200.62 port 45356 ssh2
Jul  5 07:15:35 ccis-server-5 sshd[707304]: Failed password for invalid user root from 164.92.200.62 port 47148 ssh2
Jul  5 07:15:43 ccis-server-5 sshd[707323]: Failed password for invalid user root from 164.92.200.62 port 48940 ssh2
Jul  5 07:15:54 ccis-server-5 sshd[707331]: Failed password for invalid user root from 164.92.200.62 port 50732 ssh2
Jul  5 07:16:03 ccis-server-5 sshd[707348]: Failed password for invalid user root from 164.92.200.62 port 52524 ssh2
Jul  5 07:16:13 ccis-server-5 sshd[707358]: Failed password for invalid user root from 164.92.200.62 port 54316 ssh2

.
.
.

Jul  9 02:20:39 ccis-server-5 sshd[1107903]: Failed password for invalid user gnapp from 164.92.200.62 port 47872 ssh2
Jul  9 02:21:01 ccis-server-5 sshd[1107926]: Failed password for invalid user gnapp from 164.92.200.62 port 49654 ssh2
Jul  9 02:21:27 ccis-server-5 sshd[1107969]: Failed password for invalid user gnats from 164.92.200.62 port 51436 ssh2
Jul  9 02:21:41 ccis-server-5 sshd[1107983]: Failed password for invalid user gnats from 164.92.200.62 port 53218 ssh2
Jul  9 02:22:00 ccis-server-5 sshd[1108025]: Failed password for invalid user gnats from 164.92.200.62 port 55000 ssh2

```

There is log, file size is more than 3MB

### Solution

program a watchdog to block abnormal login

If you want to run as a service <a target="_blank" href="./service.md"> LINK </a> don't follow below steps



#### Manually Run

who
    -> ```root```

pwd 
    -> ```/root/security```

ls -al 
    ->
 ```
total 24
drwxr-xr-x 2 root root 4096 Jul  9 15:38 ./
drwx------ 6 root root 4096 Jul  9 15:38 ../
-rwxr--r-- 1 root root   19 Jul  9 15:36 blacklist*
-rwxr--r-- 1 root root  980 Jul  9 15:43 log10*
-rwxr--r-- 1 root root 1261 Jul  9 15:38 ssh_watchdog.py*
-rwxr--r-- 1 root root   41 Jul  9 15:37 whitelist*
```

##### NOTICE: blacklist is blocked IP list, If you want to unblock IP, you need to cut it, and paste to whitelist
syntax may seems like ```sshd: xxx.xxx.xxx.xxx``` you need cut and paste it fully, don't missing ```sshd: ```


#### main program

```python
import os
import re

ip = []
pattern = re.compile(r'(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})')# IP pattern
get = "tail -n 100 /var/log/auth.log | grep -i 'failed password' | tail -n 10 > /root/security/log10" # get last 10 line login failed log
os.system(get)# push request

f = open('/root/security/log10','r')#open log file
line1 = f.readlines()# read log file 
f.close()
for line in line1: # read log by line
    ip.append(pattern.search(line)[0])# indentify ip

if ip: # can catch any IP
    sl = "sshd: " + ip[0] + "\n" # target IP + \n

    if ip.count(ip[0]) >= 5: # if more than 5 login failed
        f1 = open('/root/security/blacklist','r') # read blacklist
        line2 = f1.readlines()
        f1.close()
        flag=0
        if line2: # list isn't empty
            for line in line2:
                if sl==line: # IP already in blacklist
                    flag=1
                    break
        if flag==0: #  IP not in blacklist
            f2 = open('/root/security/whitelist','r') # open whitelist
            line3 = f2.readlines() # read whitelilst
            f2.close()

            flag1=0
            if line3:
                for line in line3:
                    if line == sl:
                        flag1=1 # IP in whitelist
                        break
            if flag1==0: # IP not in whitelist, block it
                sl = "sshd: " + ip[0] # remove \n
                os.system("echo " + sl + " >> /root/security/blacklist")
os.system("cat /root/security/blacklist > /etc/hosts.deny") # write to system file

```

#### do it respectably

append in <a href="https://blog.gtwang.org/linux/linux-crontab-cron-job-tutorial-and-examples/"> crontab </a> and save it

```sh
crontab -e
```

```sh
*/1 * * * * cd /root/security && python3 /root/security/ssh_watchdog.py
```

