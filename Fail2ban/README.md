# Fail2ban 

## A great way to prevent password trying attach, you can set some condition that if reached, this program will ban attach IP


### Package installation
```sh
sudo apt-get update
sudo apt-get install fail2ban
```

### Configuration

This is an example that will prevent SSH password trying attack

```sh
sudo vim /etc/fail2ban/jail.local
```

```sh
[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log

maxretry = 3
findtime = 600
bantime  = 1200
```

```sh
sudo service fail2ban restart
```


### Client management

Status:
```sh
sudo fail2ban-client status sshd
```

Show banned IP:
```sh
sudo fail2ban-client get sshd banip
```

Manual ban IP:
```sh
sudo fail2ban-client set sshd banip IP_ADDR
```

Unban IP:
```sh
sudo fail2ban-client set sshd unbanip IP_ADDR
```

Whitelist:
```sh
sudo fail2ban-client set sshd addignoreip IP_ADDR
```

