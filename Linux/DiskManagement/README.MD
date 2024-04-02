# Resize LVM in linux (Debian 12) Virtual machine


## 1º Resize the VM disk on the Virtualizer

In this case we added 30Gb to the current disk, that as all the LVMs.

## 2º List the current disk configuration

Run the following command:
Check /dev/sda partions
```
$ sudo fdisk -l /dev/sda
```
In this case, this is the following disk setup:


```
Disk /dev/sda: 80 GiB, 85899345920 bytes, 167772160 sectors
Disk model: Virtual disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x3e718d95

Device     Boot   Start       End   Sectors  Size Id Type
/dev/sda1  *       2048   1075199   1073152  524M 83 Linux
/dev/sda2       1077246 104855551 103778306 49.5G  5 Extended
/dev/sda5       1077248 104855551 103778304 49.5G 8e Linux LVM

```
We have a 50GB Extended partition with the LVM, bt the disk size is 80GB
The partition that should be deleted is the /dev/sda2


## 3º Delete de Extended partition that contains de LVM, to recreate with the rigth size:

```
$ sudo fdisk /dev/sda
```
Type 'd', the command to delete partions, and select nº '2'  to delete the /dev/sda2
This deletes de /dev/sda2 and the associated LVM /dev/sda5

```
This should be the output:

Command (m for help): d
Partition number (1,2,5, default 5): 2

Partition 2 has been deleted.
```

Then, type 'p' to see the current partion table:

```
Command (m for help): p

Disk /dev/sda: 80 GiB, 85899345920 bytes, 167772160 sectors
Disk model: Virtual disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x3e718d95

Device     Boot Start     End Sectors  Size Id Type
/dev/sda1  *     2048 1075199 1073152  524M 83 Linux
```

## 4º Recreate the partions:

First we need to create the new Extended partion with the new correct size.

Type 'n' and select 'e' for a new extended partion. Select the apropriate number, in this case is '2'.
Set the first and last sector. In this case are the default values.

```
Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): e
Partition number (2-4, default 2): 2
First sector (1075200-167772159, default 1075200):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1075200-167772159, default 167772159):

Created a new partition 2 of type 'Extended' and of size 79.5 GiB.

```

Now, type 'n' again to recreate the LVM. It will automatically create a new Logical Volume.
Set the first and last sector. It will ask If we want to remove the LVM signature, type 'N'

```
Command (m for help): n
All space for primary partitions is in use.
Adding logical partition 5
First sector (1077248-167772159, default 1077248):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1077248-167772159, default 167772159):

Created a new partition 5 of type 'Linux' and of size 79.5 GiB.
Partition #5 contains a LVM2_member signature.

Do you want to remove the signature? [Y]es/[N]o: N

```

After the creation, list the partition table with the 'p' command and this should be the output:

```
Command (m for help): p

Disk /dev/sda: 80 GiB, 85899345920 bytes, 167772160 sectors
Disk model: Virtual disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x3e718d95

Device     Boot   Start       End   Sectors  Size Id Type
/dev/sda1  *       2048   1075199   1073152  524M 83 Linux
/dev/sda2       1075200 167772159 166696960 79.5G  5 Extended
/dev/sda5       1077248 167772159 166694912 79.5G 83 Linux

```
Has we can  see, the partions have the correct size.

## 5º Change the type of the /dev/sda5

Now we need to chanfe the type of the /dev/sda5 partion to a LVM.
Type 't', to change the type, and the select nº '5' for /dev/sda5. Then type '8e', this is the alias for the Linux LVM partion

After this configuration this should be the output:

```
Command (m for help): p

Disk /dev/sda: 80 GiB, 85899345920 bytes, 167772160 sectors
Disk model: Virtual disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x3e718d95

Device     Boot   Start       End   Sectors  Size Id Type
/dev/sda1  *       2048   1075199   1073152  524M 83 Linux
/dev/sda2       1075200 167772159 166696960 79.5G  5 Extended
/dev/sda5       1077248 167772159 166694912 79.5G 8e Linux LVM
```
The type 'w' to write the changes to the disk and quit.


## 6º Resive the Phisical volume to accept the new size

To resize the disk type the following command:

```
$ sudo pvresize /dev/sda5
Physical volume "/dev/sda5" changed
1 physical volume(s) resized or updated / 0 physical volume(s) not resized

```

We can verify this by typing the following command, that shows the volume group info

```
$ sudo vgdisplay
 --- Volume group ---
  VG Name               debian-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  9
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                5
  Open LV               5
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               79.48 GiB
  PE Size               4.00 MiB
  Total PE              20348
  Alloc PE / Size       12668 / 49.48 GiB
  Free  PE / Size       7680 / 30.00 GiB
  VG UUID               y9N3ce-0pyP-Wlwn-SXlU-j6Kd-ywlE-BLgtoB

```
As we can see, the volume group now as the correct size, 79.48G with 30GB of free space

## 7º resize the Logical Volume

List the LV in the machine and choose the one to be resized

```
$ sudo lvdisplay
```

In this case, I want to resize the /dev/debian-vg/lv-var LV
To achieve this run the following command:

```
$ sudo lvextend -l +100%FREE /dev/debian-vg/lv-var
Size of logical volume debian-vg/lv-var changed from <11.18 GiB (2861 extents) to <41.18 GiB (10541 extents).
Logical volume debian-vg/lv-var successfully resized.

```

Check if the size of the LV as updated

```
$ sudo lvdisplay
...
  --- Logical volume ---
  LV Path                /dev/debian-vg/lv-var
  LV Name                lv-var
  VG Name                debian-vg
  LV UUID                yoHfYj-sq33-Zzdh-breD-Wbq5-KRvB-hx2gbS
  LV Write Access        read/write
  LV Creation host, time debian, 2024-03-14 12:44:59 +0000
  LV Status              available
  # open                 1
  LV Size                <41.18 GiB
  Current LE             10541
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:2
...
```

The size was upgraded to 41.18GB.

## 8º Extend the filesystem

So, with the previous steps we resized the Logical Volume but now it is necessary to extend the filesystem.
We can achieve this by running the following command:

```
$ sudo resize2fs /dev/debian-vg/lv-var
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/debian-vg/lv-var is mounted on /var; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 6
The filesystem on /dev/debian-vg/lv-var is now 10793984 (4k) blocks long.

```

## 9º Validate the configuration

To check if the sizes are correct run the following command:

```
$ df -h
Filesystem                       Size  Used Avail Use% Mounted on
udev                             1.9G     0  1.9G   0% /dev
tmpfs                            392M  712K  391M   1% /run
/dev/mapper/debian--vg-lv--root   18G  1.1G   16G   7% /
tmpfs                            2.0G     0  2.0G   0% /dev/shm
tmpfs                            5.0M     0  5.0M   0% /run/lock
/dev/sda1                        516M   59M  431M  12% /boot
/dev/mapper/debian--vg-lv--home   19G   48K   18G   1% /home
/dev/mapper/debian--vg-lv--tmp   919M   44K  856M   1% /tmp
/dev/mapper/debian--vg-lv--var    41G   11G   29G  27% /var
tmpfs                            392M     0  392M   0% /run/user/1000

```

As we can see, the /dev/mapper/debian--vg-lv--var as 41G as we just configured

Bibliografy:

https://www.redhat.com/sysadmin/resize-lvm-simple
https://www.baeldung.com/linux/extend-logical-extended-partitions-fdisk
https://theducks.org/2009/11/expanding-lvm-partitions-in-vmware-on-the-fly/
