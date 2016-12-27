# LVM
## Flow
1. [Create Images](#create-images): 建立2個空的disk images
1. [Loop Devices](#loop-devices): 把上面2個disk images掛載成loop devices
1. [PV creation](#pv-creation): 把2個loop devices各自建成physical volume
1. [VG creation](#vg-creation): 把2個physical volume合成一個volume group
1. [LV creation](#lv-creation): 從volume group切出一個logical volume
1. [FS creation](#fs-creation): 在完成的LV上建立File system
1. [LVM extend](#lvm-extend): 擴展LV的大小，並調整其上File system的大小
1. [Remove LVM](#remove-lvm): 完成LVM實驗，解除LVM，缷載loop devices，並刪除disk images

## Create Images
```console
# dd if=/dev/zero of=disk0.img bs=100M count=1
# dd if=/dev/zero of=disk1.img bs=100M count=1
```
## Loop Devices
```console
# sudo losetup -f disk0.img
# sudo losetup -f disk1.img
# losetup -a
```
## PV creation
```console
# sudo pvcreate /dev/loop{0,1}
  Physical volume "/dev/loop0" successfully created.
  Physical volume "/dev/loop1" successfully created.
# sudo pvs
  PV         VG Fmt  Attr PSize   PFree
  /dev/loop0    lvm2 ---  100.00m 100.00m
  /dev/loop1    lvm2 ---  100.00m 100.00m
```
## VG creation
```console
# sudo vgcreate EMUVG /dev/loop{0,1}
  Volume group "EMUVG" successfully created
# sudo vgs
  VG    #PV #LV #SN Attr   VSize   VFree
  EMUVG   2   0   0 wz--n- 192.00m 192.00m
```
## LV creation
```console
# sudo lvcreate --size 120M -n EMULV EMUVG
  Logical volume "EMULV" created.
# sudo lvs
  LV    VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  EMULV EMUVG -wi-a----- 120.00m
```
## FS creation
```console
# sudo mkfs.ext4 /dev/EMUVG/EMULV
# sudo mount /dev/EMUVG/EMULV /mnt
# df -Th /dev/EMUVG/EMULV
Filesystem              Type  Size  Used Avail Use% Mounted on
/dev/mapper/EMUVG-EMULV ext4  113M  1.6M  103M   2% /mnt
```
## LVM extend
```console
# sudo lvextend --size 180M /dev/EMUVG/EMULV
  Size of logical volume EMUVG/EMULV changed from 120.00 MiB (30 extents) to 180.00 MiB (45 extents).
  Logical volume EMUVG/EMULV successfully resized.
# sudo lvs
  LV    VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  EMULV EMUVG -wi-ao---- 180.00m
# sudo resize2fs /dev/EMUVG/EMULV
# df -Th /dev/EMUVG/EMULV
Filesystem              Type  Size  Used Avail Use% Mounted on
/dev/mapper/EMUVG-EMULV ext4  171M  1.6M  158M   1% /mnt
```
## Remove LVM
```console
# sudo umount /mnt
# sudo lvremove -f /dev/EMUVG/EMULV
  Logical volume "EMULV" successfully removed
# sudo vgremove EMUVG
  Volume group "EMUVG" successfully removed
# sudo pvremove /dev/loop{0,1}
  Labels on physical volume "/dev/loop0" successfully wiped.
  Labels on physical volume "/dev/loop1" successfully wiped.
# sudo losetup -d /dev/loop{0,1}
# rm disk{0,1}.img
```
