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
sudo apt-get install ufw
sudo ufw allow 4242
sudo ufw enable
```

edit `/etc/hostname` and `/etc/hosts` with jgotz42

`sudo visudo` ->
```
Defaults        maxtries=3
Defaults        badpass_message="Incorrect password attempt. Please try again."
Defaults        iolog_dir=/var/log/sudo
Defaults        tty_tickets
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```

```sh
sudo usermod -aG user42,sudo jgotz
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
crontab -e
```
add `@reboot /path/to/monitoring.sh`

```sh
sudo passwd --expire jgotz   # Expire the current password
sudo chage -M 30 -m 2 -W 7 jgotz  # Set password expiry to 30 days, minimum change interval to 2 days, and warning message 7 days before expiry
```

in `/etc/security/pwquality.conf`

```
minlen = 10                     # Minimum password length
dcredit = -1                    # At least one digit
ucredit = 1                     # At least one uppercase letter
lcredit = 1                     # At least one lowercase letter
ocredit = 0                     # No special characters
maxrepeat = 3                   # No more than 3 consecutive identical characters
```

in `/etc/pam.d/common-password`

```
password requisite pam_pwquality.so retry=3 difok=7 minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1 reject_username

```