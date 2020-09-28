# TP1 : Déploiement classique

# 0. Prérequis


**Setup de deux machines CentOS7 configurée de façon basique.**
* partitionnement

### On utilise le logiciel de virtualisation pour ajouter un nouveau disque a la machine.

* partitionner le nouveau disque avec LVM


```
[user@node1 ~]$ sudo pvcreate /dev/sdb
[sudo] password for user:
  Physical volume "/dev/sdb" successfully created.
[user@node1 ~]$ sudo pvs
  PV         VG     Fmt  Attr PSize  PFree
  /dev/sda2  centos lvm2 a--  <7.00g    0
  /dev/sdb          lvm2 ---   5.00g 5.00g
[user@node1 ~]$ sudo vgcreate data /dev/sdb
  Volume group "data" successfully created
[user@node1 ~]$ sudo vgs
  VG     #PV #LV #SN Attr   VSize  VFree
  centos   1   2   0 wz--n- <7.00g     0
  data     1   0   0 wz--n- <5.00g <5.00g
[user@node1 ~]$ sudo lvcreate -L 2G data -n data1
  Logical volume "data1" created.
[user@node1 ~]$ sudo lvcreate -l 100%FREE data -n data2
  Logical volume "data2" created.
[user@node1 ~]$ sudo lvs
  LV    VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root  centos -wi-ao----  <6.20g
  swap  centos -wi-ao---- 820.00m
  data1 data   -wi-a-----   2.00g
  data2 data   -wi-a-----  <3.00g
[user@node1 ~]$ sudo mkfs -t ext4 /dev/data/data1
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
131072 inodes, 524288 blocks
26214 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=536870912
16 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

[user@node1 ~]$ sudo mkfs -t ext4 /dev/data/data2
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
196608 inodes, 785408 blocks
39270 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=805306368
24 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
[user@node1 ~]$ sudo mkdir /srv/data1
[user@node1 ~]$ sudo mkdir /srv/data2
[user@node1 ~]$ sudo mount /dev/data/data1 /srv/data1
[user@node1 ~]$ sudo mount /dev/data/data2 /srv/data2
[user@node1 ~]$ sudo vim /etc/fstab
[user@node1 ~]$ sudo mount -av
[sudo] password for user:
/                        : ignored
/boot                    : already mounted
swap                     : ignored
/srv/data1               : already mounted
/srv/data2               : already mounted
[user@node1 ~]$ sudo umount /srv/data1
[user@node1 ~]$ sudo umount /srv/data2
[user@node1 ~]$ sudo mount -av
/                        : ignored
/boot                    : already mounted
swap                     : ignored
mount: /srv/data1 does not contain SELinux labels.
       You just mounted an file system that supports labels which does not
       contain labels, onto an SELinux box. It is likely that confined
       applications will generate AVC messages and not be allowed access to
       this file system.  For more details see restorecon(8) and mount(8).
/srv/data1               : successfully mounted
mount: /srv/data2 does not contain SELinux labels.
       You just mounted an file system that supports labels which does not
       contain labels, onto an SELinux box. It is likely that confined
       applications will generate AVC messages and not be allowed access to
       this file system.  For more details see restorecon(8) and mount(8).
/srv/data2               : successfully mounted
```

### Contenu du fichier fstab

```
/dev/data/data1 /srv/data1 ext4 defaults 0 0                                                                                          
/dev/data/data2 /srv/data2 ext4 defaults 0 0 
```
* un accès internet

### On configure le carte NAT en dhcp puis on verifie que une ip est attribué puis on verifie la connexion internet avec un curl.

```
[user@node1 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s3
TYPE="Ethernet"
BOOTPROTO="dhcp"
DEVICE="enp0s3"
ONBOOT="yes"

[user@node1 ~]$ ip a
[...]
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:2f:ba:8e brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic enp0s3
       valid_lft 86343sec preferred_lft 86343sec
    inet6 fe80::a00:27ff:fe2f:ba8e/64 scope link
       valid_lft forever preferred_lft forever
       
[user@node1 ~]$ curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```

### on fait la meme chose pour la deuxieme machine

```
[user@node2 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s3
TYPE="Ethernet"
BOOTPROTO="dhcp"
DEVICE="enp0s3"
ONBOOT="yes"

[user@node2 ~]$ ip a
[...]
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:60:66:b6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic enp0s3
       valid_lft 85754sec preferred_lft 85754sec
    inet6 fe80::a00:27ff:fe60:66b6/64 scope link
       valid_lft forever preferred_lft forever
       
[user@node2 ~]$ curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```

* un accès à un réseau local (les deux machines peuvent se `ping`) 

### On configure la carte HostOnly de la premiere machine:

```
[user@node1 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
TYPE=Ethernet
BOOTPROTO=static
NAME=enp0s8
DEVICE=enp0s8
ONBOOT=yes

IPADDR=192.168.1.11
NETMASK=255.255.255.0
```
### On configure la carte HostOnly de la deuxieme machine:

```
[user@node2 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
TYPE=Ethernet
BOOTPROTO=static
NAME=enp0s8
DEVICE=enp0s8
ONBOOT=yes

IPADDR=192.168.1.12
NETMASK=255.255.255.0
```

### On configure des routes statiques entre les machines


* les machines doivent avoir un nom 

### On modifie le nom de la machines dans le fichier hostname:

```
[user@localhost etc]$ echo 'node1.tp1.b2' | sudo tee /etc/hostname
```
```
[user@localhost etc]$ echo 'node2.tp1.b2' | sudo tee /etc/hostname
```
### puis on relance la machine pour que le nom s'applique

```
[user@node1 ~]$
```
```
[user@node2 ~]$
```


* les machines doivent pouvoir se joindre par leurs noms respectifs 
  
### On modifie le fichier hosts

Pour la machine 1
```
192.168.1.12 node2.tp1.b2


[user@node1 etc]$ ping node2.tp1.b2
PING node2.tp1.b2 (192.168.1.12) 56(84) bytes of data.
64 bytes from node2.tp1.b2 (192.168.1.12): icmp_seq=1 ttl=64 time=0.699 ms
64 bytes from node2.tp1.b2 (192.168.1.12): icmp_seq=2 ttl=64 time=0.484 ms
64 bytes from node2.tp1.b2 (192.168.1.12): icmp_seq=3 ttl=64 time=0.392 ms
64 bytes from node2.tp1.b2 (192.168.1.12): icmp_seq=4 ttl=64 time=0.394 ms
64 bytes from node2.tp1.b2 (192.168.1.12): icmp_seq=5 ttl=64 time=0.416 ms
^C
--- node2.tp1.b2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4002ms
rtt min/avg/max/mdev = 0.392/0.477/0.699/0.115 ms
``` 
Pour la machine 2
```
192.168.1.11 node1.tp1.b2

[user@node2 ~]$ ping node1.tp1.b2
PING node1.tp1.b2 (192.168.1.11) 56(84) bytes of data.
64 bytes from node1.tp1.b2 (192.168.1.11): icmp_seq=1 ttl=64 time=0.325 ms
64 bytes from node1.tp1.b2 (192.168.1.11): icmp_seq=2 ttl=64 time=0.536 ms
64 bytes from node1.tp1.b2 (192.168.1.11): icmp_seq=3 ttl=64 time=0.411 ms
64 bytes from node1.tp1.b2 (192.168.1.11): icmp_seq=4 ttl=64 time=0.399 ms
64 bytes from node1.tp1.b2 (192.168.1.11): icmp_seq=5 ttl=64 time=0.406 ms
^C
--- node1.tp1.b2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4003ms
rtt min/avg/max/mdev = 0.325/0.415/0.536/0.070 ms
``` 

* un utilisateur administrateur est créé sur les deux machines (il peut exécuter des commandes `sudo` en tant que `root`)

### On creer un utilisateur "admin" et on lui met les droit sudo
```
[user@node2 ~]$ sudo useradd admin
[user@node2 ~]$ sudo passwd admin admin
passwd: Only one user name may be specified.
[user@node2 ~]$ sudo passwd admin
Changing password for user admin.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
passwd: all authentication tokens updated successfully.
[user@node2 ~]$ sudo visudo
```
### contenu du fichier sudoers
```
admin    ALL=(ALL)    ALL
```

* vous n'utilisez QUE `ssh` pour administrer les machines

### connexion ssh fonctionnelle
```
PS C:\Users\Quent> ssh user@192.168.1.11
user@192.168.1.11's password:
Last login: Wed Sep 23 12:47:06 2020 from 192.168.1.1
[user@node1 ~]$
```
```
PS C:\Users\Quent> ssh user@192.168.1.12
user@192.168.1.12's password:
Last login: Thu Sep 24 09:10:48 2020 from 192.168.1.1
[user@node2 ~]$
```

* le pare-feu est configuré pour bloquer toutes les connexions exceptées celles qui sont nécessaires

### Tout les services sont bloqués sauf le ssh et un service dhcp

```
[user@node2 ~]$ sudo firewall-cmd --list-all
[sudo] password for user:
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```
[user@node1 ~]$ sudo firewall-cmd --list-all
[sudo] password for user:
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```


# I. Setup serveur Web

### configuration du fichier nginx

```bash
user usernginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;


events {
    worker_connections 1024;
}

http{

    server {
        listen 80;
        listen 443 ssl;
        server_name node1.tp1.b2;
        ssl_certificate     /etc/pki/tls/certs/server.crt;
        ssl_certificate_key /etc/pki/tls/private/server.key;
        index index.html;

        location / {
                return 301 /site1;
        }

        location /site1 {
                alias /srv/site1;
        }
        location /site2 {
                alias /srv/site2;
        }
    }
}
```

### Permission des dossiers site 1 et site 2

```
[user@node1 srv]$ ls -al
total 8
drwxr-xr-x.  6 root      root        58 Sep 23 15:27 .
dr-xr-xr-x. 17 root      root       224 Feb 21  2020 ..
drwxr-xr-x.  3 root      root      4096 Sep 23 12:02 data1
drwxr-xr-x.  3 root      root      4096 Sep 23 12:02 data2
dr-x--x--x.  2 usernginx usernginx   24 Sep 23 14:55 site1
dr-x--x--x.  2 usernginx usernginx   24 Sep 23 14:56 site2
```

### permission du fichier html

```
[user@node1 nginx]$ sudo ls -al /srv/site1/index.html
-rwxr-x--x. 1 usernginx usernginx 23 Sep 23 14:55 /srv/site1/index.html
[user@node1 nginx]$ sudo ls -al /srv/site2/index.html
-rwxr-x--x. 1 usernginx usernginx 23 Sep 23 14:56 /srv/site2/index.html
```

# II. Script de sauvegarde

### mise en place d'un utilisateur back up

```
[user@node1 ~]$ sudo useradd backup
```
### permission des fichiers de backup

```
[backup@node1 ~]$ ls -al
total 24
drwx------. 3 backup backup  132 Sep 23 19:38 .
drwxr-xr-x. 6 root   root     62 Sep 23 16:23 ..
-rw-------. 1 backup backup 1447 Sep 23 19:31 .bash_history
-rw-r--r--. 1 backup backup   18 Aug  8  2019 .bash_logout
-rw-r--r--. 1 backup backup  193 Aug  8  2019 .bash_profile
-rw-r--r--. 1 backup backup  231 Aug  8  2019 .bashrc
drwxr-xr-x. 3 backup backup   93 Sep 23 19:17 save
-rwxr-x---. 1 backup backup  286 Sep 23 19:30 tp1_backup.sh
-rw-------. 1 backup backup 4042 Sep 23 19:38 .viminfo


[usernginx@node1 site1]$ ls -al
total 4
drwxr-x---. 2 usernginx backup 24 Sep 23 14:55 .
drwxr-xr-x. 6 usernginx backup 58 Sep 23 15:27 ..
-rwxr-x--x. 1 usernginx backup 23 Sep 23 14:55 index.html
```

### contenu du fichier tp1_backup.sh


```
#!/bin/bash

backupdate=$(date +%Y_%m_%d_%H%M%S)

#répertoire de backup
dirbackup=/home/backup/save
nb_backup=find $dirbackup -type f | wc -l

archive_and_compress(){

        tar -zcf $dirbackup/site1-$backupdate.tar.gz /srv/site1
        tar -zcf $dirbackup/site2-$backupdate.tar.gz /srv/site2
}
```

# III. Monitoring, alerting

### après avoir effectuer la commande pour installer netdata et ouvert le port 19999 on obtient un accés au site.

```
bash <(curl -Ss https://my-netdata.io/kickstart.sh)

[user@node1 ~]$ sudo firewall-cmd --add-port=19999/tcp --permanent
success
[user@node1 ~]$ sudo firewall-cmd --reload
success
```

![](https://i.imgur.com/DjsnIo8.png)

### on met en place le systeme d'alerte sur discord

On creer un fichier de conf pour avoir une alarm sur le pourcentage d'utilisation de la ram

```
 alarm: ram_usage
 on: system.ram
lookup: average -1m percentage of used
 units: %
 every: 1m
 warn: $this > 10
 crit: $this > 15
 info: The percentage of RAM used by the system.
```


![](https://i.imgur.com/68uiXWq.png)

