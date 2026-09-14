# ⚡ 31 — Linux Cheat Sheets

## Navigation
```bash
pwd
ls -lah
cd /path
find /path -name '*.log'
locate filename
```

## Files
```bash
cp -a SRC DEST
mv SRC DEST
rm -i FILE
mkdir -p DIR
touch FILE
ln -s TARGET LINK
stat FILE
```

## Text
```bash
cat file
less file
head file
tail -F file
grep -i pattern file
grep -E 'a|b' file
awk '{print $1}' file
sort file | uniq -c
cut -d: -f1 /etc/passwd
```

## Users / Permissions
```bash
id USER
getent passwd
getent group
ls -l
chmod 640 file
chown user:group file
getfacl file
sudo -l
```

## Processes
```bash
ps aux
top
pgrep -a process
pstree
kill PID
nice -n 10 command
```

## Services
```bash
systemctl status SERVICE
systemctl start SERVICE
systemctl stop SERVICE
systemctl restart SERVICE
systemctl enable SERVICE
systemctl list-units --failed
journalctl -u SERVICE
```

## Packages
Debian/Ubuntu:
```bash
apt update
apt install PACKAGE
apt remove PACKAGE
dpkg -S /path/file
```
RHEL/Fedora:
```bash
dnf install PACKAGE
dnf update
dnf remove PACKAGE
rpm -q PACKAGE
```

## Storage
```bash
lsblk -f
blkid
df -h
df -i
du -sh PATH
findmnt
swapon --show
free -h
```

## Networking
```bash
ip addr
ip link
ip route
ip route get DEST
ip neigh
ss -tulpen
getent hosts NAME
ping -c 4 HOST
curl -v URL
```

## SSH
```bash
ssh user@host
ssh-keygen -t ed25519
ssh -v user@host
ssh-add -l
scp file user@host:/path
sftp user@host
sshd -t
sshd -T
```

## Firewall
```bash
nft list ruleset
iptables -L -n -v
ufw status verbose
firewall-cmd --state
firewall-cmd --list-all
```

## Logs
```bash
journalctl -b
journalctl -u SERVICE
journalctl -p warning
journalctl --since '1 hour ago'
journalctl -f
journalctl -k
aureport --auth
ausearch -m USER_LOGIN
```

## Scheduling
```bash
crontab -l
crontab -e
systemctl list-timers --all
```

## SOC Quick Flow
```text
ALERT
 ↓
TIME + HOST
 ↓
USER / IP
 ↓
PROCESS / PID
 ↓
FILE / SERVICE
 ↓
NETWORK
 ↓
LOG CORRELATION
 ↓
TIMELINE
 ↓
SCOPE + CONFIDENCE
```

## Safety Rules
- Use destructive storage commands only on known lab targets.
- Never expose private SSH keys or secrets.
- Validate firewall changes before closing remote sessions.
- Preserve evidence during investigations.
- Treat unusual behavior as a lead to investigate, not automatic proof of compromise.
