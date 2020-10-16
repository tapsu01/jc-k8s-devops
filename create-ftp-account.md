# Step-by-step install and Create FTP Account on Linux

1. Install vsFTPD

- CentOS 7

```sh
yum install vsftpd
```

- Ubuntu 

```sh
apt-get install vsftpd
```

- Start vsftpd

```sh
systemctl start vsftpd
```

- Allow vsftpd run when operator start

```sh
systemctl enable vsftpd
```

- Make sure vsftpd running on port 21

```sh
netstat -tulpn
```

2. Configs

File: `/etc/vsftpd/vsftpd.conf`

```conf
anonymous_enable=NO          # Disable anonymous user
local_enable=YES             # allow local user access server
write_enable=YES             # Allow local user CRUD files, change to `NO` only download
listen=YES                   # Enable IPv4
listen_ipv6=NO               # Disable IPv6

chroot_local_user=YES        # Enable chroot for local user.
allow_writeable_chroot=YES   #
```

3. Create chroot_list file

```sh
touch /etc/vsftpd/chroot_list
```

4. Create FTP account

- CentOS 7+

```sh
useradd -d /var/ftp/user1 -s /bin/bash user1
```

- Ubuntu

```sh
useradd -d /var/ftp/user1 -s /usr/sbin/nologin user1
```

`-d`: User Home Directory 
`-s`: Create shell
