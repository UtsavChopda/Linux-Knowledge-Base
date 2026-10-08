# Linux Administration Command Reference

## System
    hostnamectl
    uname -a
    cat /etc/os-release
    uptime
    whoami
    id
    date

## Filesystem
    pwd
    ls -lah
    cd
    mkdir -p
    touch
    cp -a
    mv
    rm -i
    find
    file
    stat
    du -sh
    df -h

## Users
    getent passwd
    getent group
    id user
    useradd
    usermod
    userdel
    passwd
    chage
    groupadd
    groups
    sudo -l -U user

## Permissions
    chmod
    chown
    chgrp
    umask
    getfacl
    setfacl
    namei -l

## Packages
    apt update
    apt search
    apt show
    apt install
    apt remove
    apt purge
    apt autoremove
    dpkg -l
    dpkg -S

## Processes
    ps aux
    ps -ef
    pstree
    pgrep
    top
    free -h
    vmstat
    kill
    nice
    renice

## Services
    systemctl status
    systemctl start
    systemctl stop
    systemctl restart
    systemctl enable
    systemctl disable
    systemctl --failed
    journalctl -u
    journalctl -b

## Networking
    ip addr
    ip link
    ip route
    ip neigh
    ss -tulpn
    ping
    traceroute
    dig
    nslookup
    curl
    nmcli

## SSH
    ssh
    ssh-keygen
    ssh-copy-id
    scp
    sftp

## Storage
    lsblk -f
    blkid
    findmnt
    df -hT
    df -ih
    du
    fdisk -l
    pvs
    vgs
    lvs
    mount
    umount

## Backup
    tar -czf
    tar -tzf
    tar -xzf
    sha256sum

## Bash validation
    bash -n script.sh
    bash -x script.sh

Never run a destructive storage/filesystem command until the target device or path has been independently verified.
