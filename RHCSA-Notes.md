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
    + NFS
        > server20:/common /dir nfs _netdev 0 0
- Repo Example
    + name= / baseurl=file:///mnt/BaseOS / gpgcheck=0
- File Systems
    + VDO
    + Stratis
    + NFS
        > Manual
        > AutoFS
    + LV
        > Physical Volume
    + FS
    + Swap
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
- YUM
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
    + Tuning Profile
    + Chrony
    + Find
        > exec - "-exec ls -ld {} \;"
    + Grep
    + Links