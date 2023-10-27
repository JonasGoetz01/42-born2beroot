# 42-born2beroot

## Login Data
- root : Password123.
- install : Password123.
- LVM : Password123.

## preparation
- download Debian DVD image
- Create VM 
- 4096 MB RAM
- 6 CPU
- 100% Video memory
- Bootorder 1. disk
- create virtual disk 50GB
- in settings create portforward 4242 4242 TCP

## install
- max storage
- guided lvm
- separate
- don't scan another installation media
- use mirror
- Germany
- deb.debian.org
- no proxy
- don't participate
- uncheck all software
- install grub
- use `/dev/sda`
- reboot


- with `root` user:

- in `/etc/apt/sources.list` remove line starting with cdrom

Take snapshot of VM

```sh
apt update && apt full-upgrade -y && apt full-upgrade -y
apt install openssh-server -y
systemctl enable ssh
```

in `/etc/ssh/sshd_config`
- change #Port 22 -> Port 4242
- add `PermitRootLogin no`
- `service sshd restart`

```sh
apt-get install sudo
usermod -aG sudo install
```

-in `/etc/sudoers` user ALL=(ALL) ALL

-from now on use ssh with iTerm

```sh
sudo apt-get update
sudo apt-get install ufw -y
sudo ufw allow 4242
sudo ufw enable
ufw status
```

edit `/etc/hostname` and `/etc/hosts` with jgotz42

`sudo visudo` ->
```
Defaults        log_input, log_output
Defaults        logfile=/var/log/sudo/sudo.log
Defaults        passwd_tries=3
Defaults        badpass_message="Incorrect password attempt. Please try again. (from jgotz)"
Defaults        iolog_dir=/var/log/sudo
Defaults        tty_tickets
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```

```sh
sudo chmod 777 /var/log/sudo/sudo.log
sudo chmod 777 /var/log/sudo
```

```sh
sudo adduser jgotz
```
- password: root
- leave all empty

```sh
sudo addgroup user42
```

```sh
sudo usermod -aG user42,sudo jgotz
```

```sh
sudo passwd --expire jgotz
```

in `/etc/login.defs`
```
PASS_MAX_DAYS   30
PASS_MIN_DAYS   2
PASS_WARN_AGE   7
```

```sh
for user in $(cut -d: -f1 /etc/passwd); do
    sudo chage -M 30 -m 2 -W 7 $user
done
```

- check: `sudo chage -l username`

```sh
sudo apt install libpam-pwquality -y
dpkg -l | grep libpam-pwquality
sudo nano /etc/pam.d/common-password
```

change line 25:
```
password requisite pam_pwquality.so retry=3
```
to
```
password requisite pam_pwquality.so retry=3 minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```

```sh
sudo passwd --expire jgotz
passwd
```

Password: NextTest1.

```sh
sudo apt-get install -y net-tools
```

create `monitoring.sh`

```
#!/bin/bash

while true; do
    WALL_MSG=$(cat <<-EOF
        #Architecture: $(uname -a)
        #CPU physical: $(nproc)
        #vCPU: $(grep -c ^processor /proc/cpuinfo)
        #Memory Usage: $(free -m | awk 'NR==2{printf "%s/%sMB (%.2f%%)", $3,$2,$3*100/$2 }')
        #Disk Usage: $(df -h / | awk 'NR==2{printf "%s/%s (%s)", $3,$2,$5}')
        #CPU load: $(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
        #Last boot: $(who -b | awk '{print $3,$4}')
        #LVM use: $(if [ -e /dev/mapper ]; then echo "yes"; else echo "no"; fi)
        #Connections TCP: $(ss -t -a | grep ESTAB | wc -l)
        #User log: $(who | wc -l)
        #Network: IP $(hostname -I | awk '{print $1}') (MAC $(ip link show | awk '/ether/ {print $2}'))
        #Sudo: $(grep -c 'COMMAND' /var/log/sudo/io/*)
EOF
)
    echo "$WALL_MSG" | wall
    sleep 600  # Sleep for 10 minutes
done

```

```sh
chmod +x monitoring.sh
```

```sh
sudo visudo
```
add
```
jgotz   ALL=(ALL) NOPASSWD: /usr/local/bin/monitoring.sh
```

```sh
sudo crontab -u root -e
```
add `*/10 * * * * /home/jgotz/monitoring.sh`


# Eval
- check partitions: `lsblk`

Aptitude vs. apt
- apt is a command line interface to manage software
- aptitude is a visual interface
- aptitude is able to fix package conflicts and show the changelogs of all packages, apt not

- check groups and user
  - `groups jgotz`

create new account:
- `sudo adduser <username>`

check firewall:
- `sudo ufw status`

hostname:
- `cat /etc/hostname && cat /etc/hosts`

- monitoring script
  - explanation:

  - interrupt:
    - `sudo crontab -u root -e`


TODO:
- The following rule does not apply to the root password: The password must have
at least 7 characters that are not part of the former password.


Bonus:
	- install Wordpress:
```sh
sudo apt update
sudo apt upgrade
sudo apt install mariadb-server -y
sudo mysql_secure_installation
```

```sh
sudo apt install php php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip -y
sudo apt install lighttpd -y
sudo nano /etc/lighttpd/lighttpd.conf
```

uncomment:
```
server.modules += ( "mod_fastcgi", "mod_rewrite" )
```
set this:
```
server.document-root        = "/var/www/html"
index-file.names            = ( "index.php", "index.html",
                                "index.htm", "default.htm",
                                "index.lighttpd.html" )

fastcgi.server = ( ".php" =>
                   (( "bin-path" => "/usr/bin/php-cgi",
                      "socket" => "/tmp/php.socket" ))
                 )
```

```sh
sudo mysql -u root -p
```

```sql
CREATE DATABASE wordpressdb;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpressdb.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

```sh
cd /tmp
sudo apt install wget -y
wget https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
sudo mv /tmp/wordpress/* /var/www/html/
```

```sh
cd /var/www/html/
sudo apt install php-cgi -y
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```

```
define('DB_NAME', 'wordpressdb');
define('DB_USER', 'wordpressuser');
define('DB_PASSWORD', 'password');
define('DB_HOST', 'localhost');
```

setup redis cache:
```sh
sudo apt install redis-server php-redis -y
sudo systemctl enable redis-server
sudo systemctl start redis-server
```
in `/var/www/html/wp-config.php`
```
define('WP_CACHE', true);
define('WP_REDIS_HOST', '127.0.0.1');
define('WP_REDIS_PORT', 6379);
```

```sh
sudo systemctl restart lighttpd
sudo systemctl restart redis-server
```

wordpress username:
- admin
- Password123.

check redis:
- `redis-cli monitor`


```sh
sudo apt install curl -y
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```