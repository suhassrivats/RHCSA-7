# RedHat Practice

**Q0: Configure Network **

```
Configure the network as follows:
- The IP address of your system should be : 172.25.X.10/24
- Subnet Mask : 255.255.255.0 => /24
- Name Server: 172.25.254.254
- Gateway: 172.25.X.254
- Note : X is your foundation number
```

**Solution:**

<u>*Note:*</u> Use console only for Password reset and Network config. For all the other operations, use terminal as it is much faster.

```
- Usually, connection is given. In such a case, no need to type MAC address
- If no connection, create a new connection. Profile name, MAC address of NIC.
- Use nmtui tool
- From base machine, ssh -X root@<name-of-vm/IP address>

```

---



**Q1: CONFIGURE SELINUX**

```
Configure the selinux mode of your system as enforcing.
```

**Solution:  **

First, set to enforcing using `setenforce` command. This is only a temporary change. Meaning, will be changed after reboot.

```bash
[root@server2 ~]# setenforce enforcing
[root@server2 ~]# getenforce
Enforcing
```

To permanently change it, edit `/etc/selinux/config` file and type `SELINUX=enforcing`

```bash
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

# Reboot after config

```

---



**Q2: CONFIGURE YUM**

```
Configure your machine such that you are able to download exam softwares from http://content.example.com/rhel7.0/x86_64/dvd/
```

**Solution:  **

1. Create a file with `.repo` extension in `/etc/yum.repos.d/` folder. In this example, I have created a repo named `errata.repo` with the given link. 

   *Note*: Make sure that there is no spaces between []. For example [abc def] is incorrect and [abc-def] is correct. In this example, it is `[Packages]`

```bash
[root@server2 yum.repos.d]# pwd
/etc/yum.repos.d

[root@server2 yum.repos.d]# ls
errata.repo

[root@server2 yum.repos.d]# cat errata.repo
[Packages]
name=Redhat packages repo
baseurl=http://content.example.com/rhel7.0/x86_64/dvd/
enabled=1
gpgcheck=0
```

2. Verify if the repo is available

```bash
[root@server2 yum.repos.d]# yum repolist
Loaded plugins: langpacks
repo id                repo name                           status
Packages               Redhat packages repo                4,305
repolist: 4,305
```

3. Try to install some packages

```bash
[root@server2 yum.repos.d]# yum install sys*
Loaded plugins: langpacks
Package systemtap-sdt-devel-2.4-14.el7.x86_64 already installed and latest version
```

---



**Q3: CONFIGURE NTP** 

```
Configure your machine to be a NTP client of classroom.example.com
```

**Solution:**

Check if NTP is enabled. If not, enable it.

```bash
# Check if NTP is enabled
[root@server2 ~]$ timedatectl
      Local time: Mon 2018-08-20 00:09:56 IST
  Universal time: Sun 2018-08-19 18:39:56 UTC
        Timezone: Asia/Kolkata (IST, +0530)
     NTP enabled: yes
NTP synchronized: no
 RTC in local TZ: no
      DST active: n/a

# If not, set-ntp as true
[ldapuser2@server2 ~]$ timedatectl set-ntp true
```

Edit `/etc/chrony.conf` file and add `server classroom.example.com iburst` line:

```bash
[root@server2 auto.master.d]# vi /etc/chrony.conf
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
#server 0.rhel.pool.ntp.org iburst
#server 1.rhel.pool.ntp.org iburst
#server 2.rhel.pool.ntp.org iburst
#server 3.rhel.pool.ntp.org iburst
server classroom.example.com iburst

# Restart chronyd.service
[root@server2 auto.master.d]# systemctl restart chronyd.service

# Check the status again. Note that, NTP is enabled
[root@server2 auto.master.d]# timedatectl
      Local time: Mon 2018-08-20 00:13:53 IST
  Universal time: Sun 2018-08-19 18:43:53 UTC
        RTC time: Sun 2018-08-19 18:43:52
        Timezone: Asia/Kolkata (IST, +0530)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a

```

---



**Q4: CREATE USERS**

```
Create the following users:
- Create a group sysadmin.
- Create a user alice who has sysadmin as a supplementary group.
- Create a user harry who also has sysadmin as his supplementary group.
- Create a user joy who does not have an interactive shell.
```

**Solution:**

```bash
# Groupadd
[root@server2 ~]# groupadd sysadmin

# Useradd to group
[root@server2 ~]# useradd alice -G sysadmin
[root@server2 ~]# useradd harry -G sysadmin
[root@server2 ~]# useradd joy -s /bin/nologin

# Add user to a group with no interactive shell
[root@server2 ~]# useradd john -s /bin/nologin -G sysadmin

# Verify
[root@server2 ~]# cat /etc/group |grep sysadmin
sysadmin:x:1001:alice,harry,john
```

---



**Q5: LVM CREATION**

```
- Create a logical volume with 20 extents where one extend is having the size of 16MiB.
- The logical volume has the name of database and volume group have the name of datastore.
- The logical volume should be mounted under the directory /mnt/database with a file system of ext3 and should be automatically available on reboot.
```

**Solution:**

```bash
# In exam 3 primary partitions will already be created. One for /, /root, logical volume.
[root@server2 ~]# lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  10G  0 disk
└─vda1 253:1    0  10G  0 part /
vdb    253:16   0  10G  0 disk
├─vdb1 253:17   0   1G  0 part
├─vdb2 253:18   0   1G  0 part
└─vdb3 253:19   0   1G  0 part

# Create a partition as an extended partition. Make sure that first and last sectors are blank
[root@server2 ~]# fdisk /dev/vdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

# Extended partition first and last sectors empty
Command (m for help): n
Partition type:
   p   primary (3 primary, 0 extended, 1 free)
   e   extended
Select (default e):
Using default response e
Selected partition 4
First sector (6293504-20971519, default 6293504):
Using default value 6293504
Last sector, +sectors or +size{K,M,G} (6293504-20971519, default 20971519):
Using default value 20971519
Partition 4 of type Extended and of size 7 GiB is set

# Create Logical volumes from the extended partition and set as type Linux-LVM
Command (m for help): n
All primary partitions are in use
Adding logical partition 5
First sector (6295552-20971519, default 6295552):
Using default value 6295552
Last sector, +sectors or +size{K,M,G} (6295552-20971519, default 20971519): +1G
Partition 5 of type Linux and of size 1 GiB is set

Command (m for help): p

Disk /dev/vdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xdd501d9b

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048     2099199     1048576   83  Linux
/dev/vdb2         2099200     4196351     1048576   83  Linux
/dev/vdb3         4196352     6293503     1048576   83  Linux
/dev/vdb4         6293504    20971519     7339008    5  Extended
/dev/vdb5         6295552     8392703     1048576   83  Linux

Command (m for help): t
Partition number (1-5, default 5):
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): p

Disk /dev/vdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xdd501d9b

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048     2099199     1048576   83  Linux
/dev/vdb2         2099200     4196351     1048576   83  Linux
/dev/vdb3         4196352     6293503     1048576   83  Linux
/dev/vdb4         6293504    20971519     7339008    5  Extended
/dev/vdb5         6295552     8392703     1048576   8e  Linux LVM

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
[root@server2 ~]# partprobe

# Check if the logical volume is created (vdb5)
[root@server2 ~]# lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  10G  0 disk
└─vda1 253:1    0  10G  0 part /
vdb    253:16   0  10G  0 disk
├─vdb1 253:17   0   1G  0 part
├─vdb2 253:18   0   1G  0 part
├─vdb3 253:19   0   1G  0 part
├─vdb4 253:20   0   1K  0 part
└─vdb5 253:21   0   1G  0 part

# Create a Physcial Volume and verify
[root@server2 ~]# pvcreate /dev/vdb5
  Physical volume "/dev/vdb5" successfully created
[root@server2 ~]# pvs
  PV         VG   Fmt  Attr PSize PFree
  /dev/vdb5       lvm2 a--  1.00g 1.00g

# Create a Volume Group and make sure that each PE is 16M (use -s option)
[root@server2 ~]# vgcreate -s 16M datastore /dev/vdb5
  Volume group "datastore" successfully created
  
[root@server2 ~]# vgs
  VG        #PV #LV #SN Attr   VSize    VFree
  datastore   1   0   0 wz--n- 1008.00m 1008.00m
  
[root@server2 ~]# vgdisplay
  --- Volume group ---
  VG Name               datastore
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               1008.00 MiB
  PE Size               16.00 MiB
  Total PE              63
  Alloc PE / Size       0 / 0
  Free  PE / Size       63 / 1008.00 MiB
  VG UUID               xYjQOl-GVa4-uzdc-w4pF-Nc8N-q1Cw-Cex65b

# Create a logical volume
[root@server2 ~]# lvcreate -n database -l 20 datastore
  Logical volume "database" created
  
[root@server2 ~]# lvs
  LV       VG        Attr       LSize   Pool Origin Data%  Move Log Cpy%Sync Convert
  database datastore -wi-a----- 320.00m
  
[root@server2 ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/datastore/database
  LV Name                database
  VG Name                datastore
  LV UUID                6jrlEs-sWZO-ZKKI-UJWJ-uQ4L-I24H-mhVH5P
  LV Write Access        read/write
  LV Creation host, time server2.example.com, 2018-08-20 23:47:29 +0530
  LV Status              available
  # open                 0
  LV Size                320.00 MiB
  Current LE             20
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           252:0

# Add the file system and mount it to directory. For permanent mount, add an entry in “/etc/fstab”
[root@server2 ~]# mkfs.ext3 /dev/datastore/database
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
81920 inodes, 327680 blocks
16384 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=67633152
40 block groups
8192 blocks per group, 8192 fragments per group
2048 inodes per group
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729, 204801, 221185

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

# Mount it to a directory
[root@server2 ~]# mount /dev/datastore/database /mnt/database/
[root@server2 ~]# cd /mnt/database/
[root@server2 database]# df -h .
Filesystem                      Size  Used Avail Use% Mounted on
/dev/mapper/datastore-database  302M  2.1M  280M   1% /mnt/database

# Find out block-id of LVM
[root@server2 database]# blkid /dev/datastore/database
/dev/datastore/database: UUID="76ee8f9e-234e-41b6-9a93-5f13c14d4b7a" TYPE="ext3"

# For permanent mount, add an entry in /etc/fstab
[root@server2 database]# tail -n1 /etc/fstab
UUID=76ee8f9e-234e-41b6-9a93-5f13c14d4b7a /mnt/database ext3 defaults 0 0

# Check if the entry is valid
[root@server2 database]# mount -a

# Reboot if no issues
[root@server2 database]# reboot

```

---



**Q6: COPY FILE**

```
Copy the file /etc/passwd to /var/tmp/hello.
The file should belong to the user root and group root.
The user alice should be able to read and write on the file.
The user harry should neither read nor write on the file.
All other users should have read permission on the file.
```

**Solution:**

```bash
# Copy the file /etc/passwd to /var/tmp/hello
[suhas@foundation2 ~]$ cp /etc/passwd /var/tmp/hello

# The file should belong to the user root and group root
[root@server2 ~]# chown root:root /var/tmp/hello
[root@server2 ~]# ls -l /var/tmp/hello
-rw-r--r--. 1 root root 2169 Aug  2 02:04 /var/tmp/hello

# User ACL restrictions
[root@server2 ~]# setfacl -m u:alice:rw /var/tmp/hello
[root@server2 ~]# setfacl -m u:harry:- /var/tmp/hello
[root@server2 ~]# setfacl -m u::--r-- /var/tmp/hello

# Verify
[root@server2 ~]# getfacl /var/tmp/hello
getfacl: Removing leading '/' from absolute path names
# file: var/tmp/hello
# owner: root
# group: root
user::rw-
user:alice:rw-
user:harry:---
group::r--
mask::rw-
```

---



**Q7: CREATE DIRECTORY**

```
Create a directory /wonderful.
The user alice and harry should be able to collaboratively work on this directory.
The files and directories created within this directory should automatically belong to the group sysadmin.
All members of the group should have read and write access. All other users should not have any permissions.
Note: By default, root user will have read and write access to all files and directories.
```

**Solution:**

```bash
# Make a directory
[root@server2 ~]# mkdir /wonderful

# Apply group permissions so that people in the group can access the directory
[root@server2 /]# chgrp sysadmin wonderful/
[root@server2 /]# ls -ld wonderful/
drwxr-xr-x. 2 root sysadmin 6 Aug  5 23:06 wonderful/

# Make sure newly created files within this directory inherits group permissions - setgid
[root@server2 /]# chmod g+s wonderful/
[root@server2 /]# ls -ld wonderful/
drwxr-sr-x. 3 root sysadmin 35 Aug  5 23:15 wonderful/

# All members of the group should have read and write access. All other users should not have any permissions.
[root@server2 /]# chmod 770 wonderful/
[root@server2 /]# ls -ld wonderful/
drwxrws---. 3 root sysadmin 35 Aug  5 23:15 wonderful/
```

---



**Q8: UPDATE KERNEL**

```
Update your kernel from http://content.example.com/rhel7.0/x86_64/errata/
```

**Solution:**

```bash
# Go to this path
[root@server2 yum.repos.d]# pwd
/etc/yum.repos.d

# Create a file with .repo extension and type repo details
[root@server2 yum.repos.d]# cat update_kernel.repo
[kernel_packages]
name=Kernel Repo
baseurl=http://content.example.com/rhel7.0/x86_64/errata/
enabled=1
gpgcheck=0

# lets go ahead and install the latest stable version of the kernel
[root@server2 yum.repos.d]# yum list kernel --showduplicates
Loaded plugins: langpacks
Installed Packages
kernel.x86_64                                          3.10.0-123.el7                                              installed
Available Packages
kernel.x86_64                                          3.10.0-123.el7                                              rhel_dvd
kernel.x86_64                                          3.10.0-123.1.2.el7                                          kernel_packages

# Check the current version of kernel
[root@server2 yum.repos.d]# uname -r
3.10.0-123.el7.x86_64

# Update the kernel. Note that the latest image from the newly created repo (kernel_packages) is picked and installed.
[root@server2 yum.repos.d]# yum update kernel
Loaded plugins: langpacks
Resolving Dependencies
--> Running transaction check
---> Package kernel.x86_64 0:3.10.0-123.1.2.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==================================================================================================================================
 Package                  Arch                     Version                                Repository                         Size
==================================================================================================================================
Installing:
 kernel                   x86_64                   3.10.0-123.1.2.el7                     kernel_packages                    29 M

Transaction Summary
==================================================================================================================================
Install  1 Package

Total download size: 29 M
Installed size: 127 M
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for kernel_packages
kernel-3.10.0-123.1.2.el7.x86_64.rpm                                                                       |  29 MB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : kernel-3.10.0-123.1.2.el7.x86_64                                                                               1/1
  Verifying  : kernel-3.10.0-123.1.2.el7.x86_64                                                                               1/1

Installed:
  kernel.x86_64 0:3.10.0-123.1.2.el7

Complete!

# Reboot the machine. We should now see the latest kernel version as default. We now see that both kernels are installed.
[root@server2 ~]# yum list kernel
Loaded plugins: langpacks
Installed Packages
kernel.x86_64                                         3.10.0-123.el7                                              installed
kernel.x86_64                                         3.10.0-123.1.2.el7                                          @kernel_packages

# Check which one is default. Yay! You now have the latest kernel version.
[root@server2 ~]# uname -r
3.10.0-123.1.2.el7.x86_64
```

---



**Q9: CRON JOB**

```
Alice must set a job to run every 7 minutes between 12 am and 2 am every day and the job is /bin/echo hi
```

**Solution:**

```bash
# Type crontab to open cron file
[root@server2 ~]# crontab -eu alice

# Add this line in the crontab file
*/7 0-2 * * * /bin/echo hi >> /etc/myfirstcron
```

---



**Q10: CREATE USER WITH SPECIFIC UID**

```
Create a user ipsr with used id 3345.
```

**Solution:**

```bash
# Create a user with specific uid
[root@server2 ~]# useradd ipsr -u 3345

# Verify user's uid
[root@server2 ~]# cat /etc/passwd |grep ipsr
ipsr:x:3345:3345::/home/ipsr:/bin/bash
```

---



**Q11: SWAP PARTITION**

```
Create a swap partition of 256M and should be automatically available on reboot.
```

**Solution:**

```bash
# Create a swap using extended partition
[root@server2 ~]# fdisk /dev/vdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help):
Command (m for help): n
All primary partitions are in use
Adding logical partition 6
First sector (8394752-20971519, default 8394752):
Using default value 8394752
Last sector, +sectors or +size{K,M,G} (8394752-20971519, default 20971519): +1G
Partition 6 of type Linux and of size 1 GiB is set

Command (m for help): t
Partition number (1-6, default 6): 82
Value out of range.
Partition number (1-6, default 6):
Hex code (type L to list all codes): 82
Changed type of partition 'Linux' to 'Linux swap / Solaris'

Command (m for help): p

Disk /dev/vdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xdd501d9b

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048     2099199     1048576   83  Linux
/dev/vdb2         2099200     4196351     1048576   83  Linux
/dev/vdb3         4196352     6293503     1048576   83  Linux
/dev/vdb4         6293504    20971519     7339008    5  Extended
/dev/vdb5         6295552     8392703     1048576   8e  Linux LVM
/dev/vdb6         8394752    10491903     1048576   82  Linux swap / Solaris

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
[root@server2 ~]# partprobe
[root@server2 ~]# lsblk
NAME                   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda                    253:0    0   10G  0 disk
└─vda1                 253:1    0   10G  0 part /
vdb                    253:16   0   10G  0 disk
├─vdb1                 253:17   0    1G  0 part
├─vdb2                 253:18   0    1G  0 part
├─vdb3                 253:19   0    1G  0 part
├─vdb4                 253:20   0    1K  0 part
├─vdb5                 253:21   0    1G  0 part
│ └─datastore-database 252:0    0  912M  0 lvm  /mnt/database
└─vdb6                 253:22   0    1G  0 part

# Create swap space using mkswap
[root@server2 ~]# mkswap /dev/vdb6
Setting up swapspace version 1, size = 1048572 KiB
no label, UUID=9375bdb2-9b30-48ac-9508-4f24220a6459

# Add an entry in /etc/fstab and verify
[root@server2 ~]# tail -n1 /etc/fstab
UUID=9375bdb2-9b30-48ac-9508-4f24220a6459 swap swap defaults 0 0
[root@server2 ~]# mount -a

# Swapon
[root@server2 ~]# swapon -a

# Verify
[root@server2 ~]# swapon -s
Filename                                Type            Size    Used    Priority
/dev/vdb6                               partition       1048572 0       -1

```

---



**Q12: LOCATE FILES**

```
Find all files owned by the user root and group mail, copy the output files to /root/findings.
```

**Solution:**

```bash
# Create a destination directory if it does not exist
[root@server2 /]# mkdir -p /root/findings

# Find all files owned by the user root and group mail
[root@server2 /]# find / -user root -group mail
find: ‘/proc/29915/task/29915/fd/6’: No such file or directory
find: ‘/proc/29915/task/29915/fdinfo/6’: No such file or directory
find: ‘/proc/29915/fd/6’: No such file or directory
find: ‘/proc/29915/fdinfo/6’: No such file or directory
/var/spool/mail

# Find all files owned by the user root and (-a option) group mail, copy the output files to /root/findings
[root@server2 /]# find / -user root -a -group mail -exec cp -r {} /root/findings \;
find: ‘/proc/29947/task/29947/fd/6’: No such file or directory
find: ‘/proc/29947/task/29947/fdinfo/6’: No such file or directory
find: ‘/proc/29947/fd/6’: No such file or directory
find: ‘/proc/29947/fdinfo/6’: No such file or directory

# Make sure the files are copied the destination directory
[root@server2 findings]# pwd
/root/findings
[root@server2 findings]# ll
total 0
drwxr-xr-x. 2 root root 75 Aug  6 00:51 mail
[root@server2 findings]# cd mail/
[root@server2 mail]# ll
total 0
-rw-r-----. 1 root root 0 Aug  6 00:51 alice
-rw-r-----. 1 root root 0 Aug  6 00:51 harry
-rw-r-----. 1 root root 0 Aug  6 00:51 ipsr
-rw-r-----. 1 root root 0 Aug  6 00:51 joy
-rw-r-----. 1 root root 0 Aug  6 00:51 rpc
-rw-r-----. 1 root root 0 Aug  6 00:51 student
```

---



**Q13: SEARCH WORDS**

```
Display the matches for the words which begin with "ns" in the /usr/share/dict/words and save the output to a file /home/student/locate.txt.
```

**Solution:**

```bash
# Use ^ for starting
[root@server2 dict]# grep ^ns words > /home/student/locate.txt
[root@server2 dict]# cat /home/student/locate.txt
ns
ns-a-vis
nsec
```

---



**Q14: AUTOFS**

```
- The home directory of LDAP users are shared via NFS.
- The classroom.example.com (172.25.254.254)shares home directory of ldapusers via NFS.
- Mount /home/guests/ldapuserX to your system, where x is your foundation number.
- The ldapuserx's home directory is at classroom.example.com:/home/guests/ldapuserx.
- The ldapuserx's home directory should be automounted locally beneath /home/guests.
- The home directories must be writable by their users.
```

**Solution:**

***Note:*** LDAP should be configured before this step. Refer to **Q17**

1. Install `autofs` using `yum install autofs`

2. Create a new file in `/etc/auto.master.d` named `home.autofs`

   ```bash
   [root@server2 ~]# getent passwd ldapuser2
   ldapuser2:*:1702:1702:LDAP Test User 2:/home/guests/ldapuser2:/bin/bash
   
   [root@server2 auto.master.d]# vi home.autofs
   [root@server2 auto.master.d]# cat home.autofs
   /home/guests /etc/auto.home
   ```

3. Update `/etc/auto.home` with mount-point information from a different server and directory

   ```bash
   [root@server2 auto.master.d]# cat /etc/auto.home
   ldapuser2 -rw,sync,nfsvers=3 classroom.example.com:/home/guests/ldapuser2
   ```

4. Start  `autofs` service and verify if it is active

   ```bash
   [root@server2 auto.master.d]# systemctl start autofs
   
   [root@server2 auto.master.d]# systemctl status autofs
   autofs.service - Automounts filesystems on demand
      Loaded: loaded (/usr/lib/systemd/system/autofs.service; disabled)
      Active: active (running) since Sun 2018-06-24 02:10:20 IST; 7s ago
     Process: 30292 ExecStart=/usr/sbin/automount $OPTIONS --pid-file /run/autofs.pid (code=exited, status=0/SUCCESS)
    Main PID: 30294 (automount)
      CGroup: /system.slice/autofs.service
              └─30294 /usr/sbin/automount --pid-file /run/autofs.pid
   
   Jun 24 02:10:20 server2.example.com systemd[1]: Starting Automounts filesystems on demand...
   Jun 24 02:10:20 server2.example.com automount[30294]: setautomntent: lookup(sss): setautomntent: No such file or directory
   Jun 24 02:10:20 server2.example.com systemd[1]: Started Automounts filesystems on demand.
   ```

5. ssh as `ldapuser2` into `localhost` and verify

   ```bash
   [root@server2 auto.master.d]# ssh ldapuser2@localhost
   The authenticity of host 'localhost (::1)' can't be established.
   ECDSA key fingerprint is 65:4d:ac:8a:c9:58:82:b5:0c:91:c4:ef:a5:e6:f6:65.
   Are you sure you want to continue connecting (yes/no)? yes
   Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
   ldapuser2@localhost's password: 
   
   [ldapuser2@server2 ~]$ pwd
   /home/guests/ldapuser2
   
   [ldapuser2@server2 ~]$ df -h .
   Filesystem                                    Size  Used Avail Use% Mounted on
   classroom.example.com:/home/guests/ldapuser2   10G  3.7G  6.4G  37% /home/guests/ldapuser2
   ```

---



**Q15: LVMRESIZE**

```
Resize the logical volume 'm7_storage' to 900M which belongs to the volume group 'vgroup'.

# Note: For this example I am choosing 'database' LVM.
```

**Solution:**

```bash
# Check the LVM using lvs command and resize
[root@server2 ~]# lvresize -r -L 900M /dev/datastore/database
  Rounding size to boundary between physical extents: 912.00 MiB
  Extending logical volume database to 912.00 MiB
  Logical volume database successfully resized
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/mapper/datastore-database is mounted on /mnt/database; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 4
The filesystem on /dev/mapper/datastore-database is now 933888 blocks long.

# Verify if the mount-point for this LVM is updated as well.
[root@server2 ~]# df -h /mnt/database/
Filesystem                      Size  Used Avail Use% Mounted on
/dev/mapper/datastore-database  876M  2.6M  830M   1% /mnt/database



```

---



**Q16: BACKUP FILES**

```
Create a bzip2 archive /root/today_backup.tar.bz2 which stores the backup of /etc .
```

**Solution:**

```bash
# Tar using bzip2 (j)
[root@server2 etc]# tar -jcf /root/today_backup.tar.bz2 /etc
tar: Removing leading `/' from member names

# Verify if the tarred file exists
[root@server2 etc]# ls -ld /root/today_backup.tar.bz2
-rw-r--r--. 1 root root 7309242 Aug  7 00:05 /root/today_backup.tar.bz2

[root@server2 etc]# cd /root/
[root@server2 ~]# ll
total 7152
-rw-------. 1 root root    8619 May  7  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root 7309242 Aug  7 00:05 today_backup.tar.bz2
```

---



**Q17: ACCESS NETWORK USERS**

```
- Bind your system to the LDAP server provided at classroom.example.com
- The base DN is `dc=example,dc=com`.
- You can download the TLS certificate from <http://classroom.example.com/pub/example-ca.crt>
- Use LDAP password for authentication and obtaining user information.
- Log in as ldapuserX, (where X is your foundation number) with password 'password'.
```

**Solution:  **

1. Install `authconfig-gtk` and `sssd` packages using `yum -y install authconfig-gtk sssd`.

2. You will get a pop-up window, fill all LDAP details in the window and apply the changes.

   ![Authconfig Popup Window](Redhat Images/authconfig-gtk.png)

3. Verify if the user is added to LDAP.

   ```bash
   [root@server2 ~]# getent passwd ldapuser2
   ldapuser2:*:1702:1702:LDAP Test User 2:/home/guests/ldapuser2:/bin/bash
   ```

