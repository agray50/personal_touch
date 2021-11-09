- Networking
    + nmcli
    + manual
- Mounting Examples
    + FS
        > /dev/vgfs/ext4vol /dir ext4 defaults 0 0
    + VDO
        > /dev/mapper/vdo1 /dir xfs x-systemd.requires=vdo.service 0 0
    + UUID
        > UUID="xxxxxxxx" /dir ext4 defaults 0 0
    + Local Disk
        > /dev/sr0 /mnt iso9660 r0 0 0
    + Stratis
        > UUID="xxxxxxxx" /dir xfs x-systemd.requires=stratisd.service 0 0
    + NFS
        > server20:/common /dir nfs _netdev 0 0
    + Swap
        > UUID="xxxxxxxx" swap swap pri=1 0 0
- Repo Example
    + name= / baseurl=file:///mnt/BaseOS / gpgcheck=0 | yum repolist
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
        > echo "autoindir {ServerIP}:/dir" >> /etc/auto.misc
        > systemctl restart autofs
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
    + Swap
        > parted /dev/sdd mklabel msdos | mkpart
        > mkswap -L sdd3 /dev/sdd1 40 #Partition
        > vgcreate vgfs /dev/sdd1 | lvcreate vgfs --name swapvol -L 132
        > mkswap /dev/vgfs/swapvol
- User/Group
    + User
        > Passwd - "cd /etc ; grep user1: passwd shadow group gshadow"
    + Group
    + Permissions
    + Advanced
        > Setuid - u+s #Executed by non owners with owner permissions
        > Setgid - g+s #Inherit Group Owner
        > Sticky Bits - o+t #Protects a files and subdirectories from being deleted or moved
- ACLs
    + setfacl - "-m u:user100:rw filename" "-x u:user100 filename" "-b filename {Remove All}"
    + getfacl - "-c"
    + defaultacl - "setfacl -dm u:user100:rw, u:user200:rw /dir" #For consistent dir perms
- Boot
    + Reset Root Password
        > #add rd.break after "rhgb quiet"
        > mount -o remount, rw /sysroot | chroot /sysroot | passwd | touch /.autorelabel
- YUM
    + Individual
        > config-manager
        > list
        > info
    + Groups
        > group list
        > group info
    + Modules
        > module list
        > module info
        > module reset perl
        > install perl:5.26/minimal --allowerasing
- Firewalld
- SELinux
    + semanage | fcontext / port
    + setsebool / getsebool
    + chcon
- Bash
    + Integers
        > eq | equal, ne | not equal, gt | greater than, ge | greater than or eq, lt | less than, le | less than or eq
    + Strings
        > = | equal, != | not equal, \< | less than, \> | greater than
- Podman
- SSH
    + 
- Misc
    + Tar
    + Environment Variables
    + Jobs
        > At - at 11:30pm 03/31/2021 | atrm 1 | at -c 1
        > Cron - crontab | */5 10,11 5,20 ** echo "Hello World" #/etc/cron.allow/deny for permissions
    + Tuning Profile
        > tuned-adm profile | recommend | {type} | off
    + Chrony
    + Find
        > exec - "-exec ls -ld {} \;"
    + Grep
    + Links
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