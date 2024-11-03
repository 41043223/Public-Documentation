## GeoIP

GeoIP can help us to know a specify IP from which country.It's usually using for block countries to access service.

Here have a GeoIP database named <a href="https://db-ip.com/db/download/ip-to-country-lite" target="_blank"> IP to Country Lite database </a> 

Ref:
<a href="https://ithelp.ithome.com.tw/articles/10312284?sc=rss.qu" target="_blank"> itHelp(IPtable) </a>
<a href="https://www.seenlyst.com/blog/geo-blocking-ufw-iptables/" target="_blank"> seenlyst.com(UFW) </a>

## Download database

1. Using apt to download needed package

```sh
sudo apt update
sudo apt-get install curl unzip perl
sudo apt-get install xtables-addons-common
sudo apt-get install libtext-csv-xs-perl libmoosex-types-netaddr-ip-perl
```

2. Make a dir for storage database

```sh
sudo mkdir /usr/share/xt_geoip
```

3. Write a script for download database

```sh
sudo vim /usr/local/bin/update-geoip.sh
```

```sh
#!/bin/bash

# Create temporary directory
mkdir -p /usr/share/xt_geoip/tmp/

# Download the geoip database file
cd /usr/share/xt_geoip/tmp/
/usr/libexec/xtables-addons/xt_geoip_dl

# Update geoip xtables
/usr/bin/perl /usr/libexec/xtables-addons/xt_geoip_build -D /usr/share/xt_geoip/

# Remove temp directory
rm -r /usr/share/xt_geoip/tmp/
```

NOTE: ```xtables-addons``` is a directory that storage ```xt_geoip_dl``` and ```xt_geoip_build```, the pwd is ```/usr/libexec/xtables-addons```


4. Run script

```sh
sudo chmod +x /usr/local/bin/update-geoip.sh
sudo /usr/local/bin/update-geoip.sh
```

5. If no error, write to crontab

```sh 
sudo crontab -e
```

```sh
0 5 15 * * /usr/local/bin/update-geoip.sh >> /var/log/update-geoip.log 2>&1
```

## UFW setting

1. Set UFW policy to default deny

```sh
sudo ufw default deny
```

2. Open your port 

```sh
sudo ufw allow PORT/TCP(UDP) comment "Comment here"
```

3. Backup Original rule

```sh
sudo cp /etc/ufw/before.rules /etc/ufw/before.rules.Backup
sudo cp /etc/ufw/before6.rules /etc/ufw/before6.rules.Backup
```

4. Edit IPv4 rules

```sh 
sudo vim /etc/ufw/before.rules
```

You can using below rules

```sh
ufw-before-input
ufw-before-output
ufw-before-forward
```

Here's my sample, Append your rule before ```COMMIT```

```sh
# Accept only TW and JP
-A ufw-before-input -p tcp --dport 22 -m geoip --src-cc TW,JP -j ACCEPT
-A ufw-before-input -p tcp --dport 80 -m geoip --src-cc TW,JP -j ACCEPT
-A ufw-before-input -p tcp --dport 443 -m geoip --src-cc TW,JP -j ACCEPT
-A ufw-before-input -p tcp --dport 25565 -m geoip --src-cc TW,JP -j ACCEPT
-A ufw-before-input -p udp --dport 19132 -m geoip --src-cc TW,JP -j ACCEPT
```

5. Edit IPv6 rules

```sh 
sudo vim /etc/ufw/before6.rules
```

You can using below rules

```sh
ufw6-before-input
ufw6-before-output
ufw6-before-forward
```

Here's my sample, Append your rule before ```COMMIT```

```sh
# Accept only TW and JP
-A ufw6-before-input -p tcp --dport 22 -m geoip  --src-cc TW,JP -j ACCEPT
-A ufw6-before-input -p tcp --dport 80 -m geoip  --src-cc TW,JP -j ACCEPT
-A ufw6-before-input -p tcp --dport 443 -m geoip  --src-cc TW,JP -j ACCEPT
-A ufw6-before-input -p tcp --dport 25565 -m geoip  --src-cc TW,JP -j ACCEPT
-A ufw6-before-input -p udp --dport 19133 -m geoip  --src-cc TW,JP -j ACCEPT
```

6. Enable and reload UFW

```sh
sudo ufw enable
sudo ufw reload
```

