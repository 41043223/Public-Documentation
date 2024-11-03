### RUN as a <a target="_blank" href="https://cfarm.blog.aznc.cc/%E4%BD%BF%E7%94%A8-systemd-timer-%E4%BB%A3%E6%9B%BF-crontab/">service</a> 

If you want to run as a service,

who ->
```root```
pwd ->
```/etc/ssh_watchdog``` 

ls -al ->
```sh
total 24
drwxr-xr-x   2 root root  4096 Jul 10 03:55 ./
drwxr-xr-x 123 root root 12288 Jul 10 03:07 ../
-rw-r--r--   1 root root    20 Jul 10 02:17 blacklist
-rwxr--r--   1 root root     0 Jul 10 03:55 log10*
-rwxr--r--   1 root root  1572 Jul 10 03:15 ssh_watchdog.py*
-rw-r--r--   1 root root     0 Jul 10 00:30 whitelist
```

##### NOTICE: blacklist is blocked IP list, If you want to unblock IP, you need to cut it, and paste to whitelilst
syntax may seems like ```sshd: xxx.xxx.xxx.xxx``` you need cut and paste it fully, don't missing ```sshd: ```


main program  ->

```py
import os
import re

ip = []
pattern = re.compile(r'(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})')# IP pattern
get = "tail -n 100 /var/log/auth.log | grep -i 'failed password' | tail -n 10 > /etc/ssh_watchdog/log10" # get last 10 line login failed log
os.system(get)# push request

f = open('/etc/ssh_watchdog/log10','r')#open log file
line1 = f.readlines()# read log file
f.close()
for line in line1: # read log by line
    ip.append(pattern.search(line)[0])# indentify ip

if ip: # can catch any IP
    sl = "sshd: " + ip[0] + "\n" # target IP + \n

    if ip.count(ip[0]) >= 5: # if more than 5 login failed
        f1 = open('/etc/ssh_watchdog/blacklist','r') # read blacklist
        line2 = f1.readlines()
        f1.close()
        flag=0
        if line2: # list isn't empty
            for line in line2:
                if sl==line: # IP already in blacklist
                    flag=1
                    break
        if flag==0: #  IP not in blacklist
            f2 = open('/etc/ssh_watchdog/whitelist','r') # open whitelist
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
                os.system("echo " + sl + " >> /etc/ssh_watchdog/blacklist")
os.system("cat /etc/ssh_watchdog/blacklist > /etc/hosts.deny") # write to system file

```


#### Service files


ssh_watchdog.service

```sh
[Unit]
Description=A script to block abnormal SSH login

[Service]
Type=simple
ExecStart=/bin/python3 /etc/ssh_watchdog/ssh_watchdog.py
```

ssh_watchdog.timer
```sh
[Unit]
Description=Run ssh_watchdog every minutes

[Timer]
OnCalendar=*-*-* *:*:00
Unit=ssh_watchdog.service

[Install]
WantedBy=multi-user.target
```
put that two files to ```/etc/systemd/system```

and enable timer ```systemctl enable ssh_watchdog.timer```