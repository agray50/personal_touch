- Networking
    + hostname
        > vi /etc/hostname | systemctl restart systemd-hostnamed
        > hostnamectl set-hostname
    + manual
        > cd /etc/sysconfig/network-scripts | cp ifcfg
        > BOOTPROTO=static | IPV6INIT=no | ONBOOT=yes | IPADDR=x.x.x.x | PREFIX=24 | GATEWAY=x.x.x.1 | DNS=x.x.x.x
        > ifdown / ifup
        > ip a
    + nmcli
        > nmcli dev status
        > nmcli con add type Ethernet ifname enp0s con-name enp0s ip4 x.x.x.x/y gw4 x.x.x.x ipv4.dns x.x.x.x
        > nmcli con show / nmcli con down / nmcli con up
        > ip a
    + hosts table
        > vi /etc/hosts
        > x.x.x.x server20.example.com server20
    + PS1
        > \H full hostname 
- Mounting Examples
    + FS
        > /dev/vgfs/ext4vol /dir ext4 defaults 0 0
    + VDO
        > /dev/mapper/vdo1 /dir xfs x-systemd.requires=vdo.service 0 0
    + UUID
        > UUID="xxxxxxxx" /dir ext4 defaults 0 0
    + Local Disk
        > /dev/sr0 /mnt iso9660 ro 0 0
    + Stratis
        > UUID="xxxxxxxx" /dir xfs x-systemd.requires=stratisd.service 0 0
    + NFS
        > server20:/common /dir nfs _netdev 0 0
    + Swap
        > UUID="xxxxxxxx" swap swap pri=1 0 0
- Repo Example
    + name= / baseurl=file:///mnt/BaseOS / gpgcheck=0 / enabled=1| yum repolist
- File Systems
    + lsblk | df -hT | /etc/fstab mount -a |
        > MBR FS UUID - lsblk -f
        > LV FS UUID - blkid
        > Stratis FS UUID - lsblk -o UUID
    + MBR
        > parted /dev/sdb print | mklabel msdos | mkpart
    + VDO
        > install vdo kmod-vdo | systemctl enable vdo.service
        > wipefs -a /dev/sdc
        > vdo create --name vdo-vol1 --device /dev/sdc --vdoLogicalSize 16G --vdoSlabSize 128
        > vdo remove --name vdo-vol1 | vdo list
    + Stratis
        > install stratis-cli | systemctl enable stratisd.service
        > stratis pool create mypool /dev/sdd
        > stratis filesystem create mypool myfs
        > stratis pool list / stratis filesystem list
        > stratis pool destroy / stratis filesystem destroy
        > mkdir /myfs1 | mount /stratis/mypool/myfs /myfs1
        > stratis pool add-data mypool /dev/sdd
        > stratis pool rename mypool mynewpool
    + Manual NFS
        > #Server
        > yum install nfs-utils
        > firewall-cmd --permanent --add-service=nfs | firewall-cmd --reload
        > systemctl enable nfs-server.service
        > echo "/dir *(rw)" >> /etc/exports
        > exportfs -a
        > #Client
        > yum install cifs-utils
        > mkdir /dir2 | chmod 755 /dir2
        > mount {ServerIP}:/dir /dir2
    + AutoFS
        > yum install autofs
        > #Direct Map
        > echo "/- /etc/auto.master.d/auto.dir" >> /etc/auto.master
        > echo "/autodir {ServerIP}:/dir" >> /etc/auto.master.d/auto.dir
        > systemctl restart autofs
        > #Indirect Map
        > echo "dir {ServerIP}:/dir" >> /etc/auto.misc
        > systemctl restart autofs
        > #Mount Directory
        > Make user on server and set password
        > /home /server(rw) | exportfs -av
        > Make user on client and set base home dir (-b) and no home dir (-M)
        > mkdir /dir1
        > vi /etc/auto.master | /dir1 /etc/auto.master.d/auto/home
        > * -rw server:/home/&
    + LV / VG
        > parted /dev/sdd mklabel msdos | mkpart | set 1 lvm on
        > pvcreate /dev/sdd1 /dev/sdb
        > vgcreate -s 16 vgbook /dev/sdd1 /dev/sdb
        > pvs / vgs / lvs | pvdisplay / vgdisplay / lvdisplay
        > lvcreate -L 120M -n lvbook1 vgbook
        > lvrename vgbook/lvbook1 vgbook/lvbook2
        > lvreduce vgbook/lvbook2 -L 50M | lvextend vgbook/lvbook2 -l +32M
        > vgreduce vgbook /dev/sdd1 /dev/sdd2
        > lvremove / vgremove / pvremove
    + FS
        > mkfs.xfs | mkfs.ext4
        > lvresize -r -L +40 /dev/vgfs/xfsvol
        > xfs_growfs
    + Swap
        > parted /dev/sdd mklabel msdos | mkpart
        > mkswap -L sdd3 /dev/sdd1 40 #Partition
        > vgcreate vgfs /dev/sdd1 | lvcreate vgfs --name swapvol -L 132
        > mkswap /dev/vgfs/swapvol
        > swapon -a
- User/Group
    + User
        > Passwd - "cd /etc ; grep user1: passwd shadow group gshadow"
        > chage #account expiry
        > useradd -s /sbin/nologin #non-interactive shell
        > usermod -L #lockaccount
    + Group
    + Permissions
        > chgrp / chmod / chown
    + Advanced
        > Setuid - u+s #Executed by non owners with owner permissions
        > Setgid - g+s #Inherit Group Owner
        > Sticky Bits - o+t #Protects a files and subdirectories from being deleted or moved
- ACLs
    + setfacl
        > setfacl -m u:user100:rw filename" "-x u:user100 filename" "-b filename {Remove All}"
    + getfacl
        > getfacl -c
    + defaultacl
        > setfacl -dm u:user100:rw, u:user200:rw /dir" #For consistent dir perms
- Boot
    + Reset Root Password
        > #add rd.break after "rhgb quiet"
        > chroot /sysroot | mount -o remount,rw / | passwd | touch .autorelabel
        > exit | reboot
    + Grub
        > vi /etc/default/grub
        > grub2-mkconfig -o /boot/grub2/grub.cfg
        > remove rhgb quiet for boot messages
    + Systemctl
        > systemctl isolate graphical
- YUM
    + Individual
        > yum config-manager
        > yum list
        > yum info
    + Groups
        > yum group list
        > yum group info
    + Modules
        > yum module list
        > yum module info
        > yum module reset perl
        > yum module install perl:5.26/minimal --allowerasing
- Firewalld
    + firewall-cmd
        > firewall-cmd --add-service=http --permanent
        > firewall-cmd --add-port=80/tcp --permanent
        > firewall-cmd --zone=internal --add-port=5901-5910/tcp --permanent
        > firewall-cmd --reload
        > firewall-cmd --list-services / --list-ports
        > cat /etc/firewalld/zones/public.xml / cat /etc/firewalld/zones/internal.xml
        > firewall-cmd --get-active-zones
        > firewall-cmd --set-default-zone=internal
        > firewall-cmd --remove-service=ssh --permanent
- SELinux
    + ll -Z | restorecon -R /tmp/dir
    + chcon
        > chcon -u user_u -t public_content_t -R /dir
    + semanage
        > semanage fcontext -a -t public_content_t -s user_u '/tmp/dir(/.*)?'
        > semanage port -a -t http_port_t -p tcp 8010 | semanage port -d -t http_port_t -p tcp 8010
        > semanage port -l
        > cp file.txt --preserve=context
    + setsebool / getsebool
        > getsebool nfs_export_all_rw
        > setsebool nfs_export_all_rw_off -P #Persistent
    + setenforce
        > setenforce permissive | vi /etc/selinux/config
- Bash
    + Integers
        > eq | equal, ne | not equal, gt | greater than, ge | greater than or eq, lt | less than, le | less than or eq
    + Strings
        > = | equal, != | not equal, \< | less than, \> | greater than
    + For Loop
        > for | do | done
    + While Loop
        > while | do | done
    + IF Loop
        > if | elif | then | else | fi
- Podman
    + Setup
        > yum module install container-tools
        > /etc/containers/registries.conf
    + Commands
        > podman search
        > podman inspect
        > podman rmi #remove image
        > podman login
        > podman port
        > -e SECRET="xxxx"
        > #Root User for Root Containers and Normal User for Rootless Containers
        > chmod 777 /dir | -v /dir:/container_data:Z
        > -p hostport:containerport
    + Systemd (Root Container)
        > podman generate systemd --new --name root-container | sudo tee /etc/systemd/system/root-container.service
        > systemctl daemon-reload
        > systemctl enable --now root-container
    + Systemd (Rootless Container)
        > mkdir ~/.config/systemd/user -p
        > podman generate systemd --new --name rootless-container > ~/.config/systemd/user/rootless-container.service
        > systemctl --user daemon-reload
        > systemctl enable --user --now rootless-container
- SSH
    + ssh
        > ssh user1@server20
        > ssh-keygen
        > ssh-copy-id
    + scp
        > sudo scp /dir1 server1:/dir2
    + rsync
        > rsync -avPz /etc/rsyslog.conf server20:/tmp
- Misc
    + Tar
        > tar -cvf / tar -xvf /tmp/etc.tar /etc/sysconfig
    + rpm
        > rpm -ql list files
        > rpm -qa list packages
    + kernel
        > uname -r
        > access.redhat.com | Downloads | Red Hat Enterprise Linux 8 by Category | Click Packages and Enter Kernel
        > Download latest for kernel packages
        > yum install /tmp/kernel*
    + journald
        > mkdir /var/log/journal
        > systemctl restart systemd-journald
    + Logger
        > logger "This is the RHCSA sample exam on $(date) by $LOGNAME"
        > grep "This is the" /var/log/messages
    + Environment Variables
        > printenv
        > $PS1
    + Jobs
        > at 11:30pm 03/31/2021 | atrm 1 | at -c 1
        > crontab | */5 10,11 5,20 ** echo "Hello World" #/etc/cron.allow/deny for permissions
        > crontab -u user70 -e #Set Cron for a user
    + Tuning Profile
        > tuned-adm profile | recommended | {type} | off
    + Boot Target
        > systemctl set-default multi-user.target
    + Chrony
        > yum install chrony
        > vi /etc/chrony.conf
        > systemctl enable chronyd.service
        > chronyc
    + Timedatectl
        > timedatectl set-timezone / timedatectl set-ntp / timedatectl status
    + Find
        > -exec ls -ld {} \;
        > find / -mtime -30 >> /var/tmp/modfiles.txt
    + Grep
        > grep '^essential' /usr/share > /tmp/pattern.txt
    + Links
        > ln file1.txt file2.txt #hardlink
        > ln -s /dir/file1.txt file2.txt #softlink
    + Set Permanent Variables
        > ~/.bashrc #Variables stored here reside in the user's home directory
        > /etc/profile #Accesable by all users and are loaded whenever a new shell is opened
        > /etc/environment #Accesable system-wide
    + Enable packet forwarding
        > vi /etc/sysctl.conf
        > net.ipv4.port_forward=1
    + Automount User Home Directory Using Indirect Map
        > #Server
        > useradd -u 3000 user30
        > echo password1 | passwd --stdin user30
        > echo "/home *(rw)" >> /etc/exports
        > exportfs -ar
        > #Client
        > useradd user30 -u 3000 -Mb /nfshome
        > echo password1 | passwd --stdin user30
        > mkdir /nfshome
        > echo "/nfshome /etc/auto.master.d/auto.home" >> /etc/auto.master
        > echo "* -rw &:/home&" >> /etc/auto.master.d./auto.home
        > systemctl enable autofs.service
        > sudo su - user30
    + Basic Web Server
        > vi /var/www/html/index.html