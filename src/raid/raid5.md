# Raid 5
## Flow
1. [Create Images](#create-images): 建立4個空的disk images
2. [Loop Devices](#loop-devices): 用上述4個disk images建立loop devices
3. [Raid 5](#raid-5): 將4個loop devices其中3個組成raid 5裝置/dev/md0，另外一個當做備援硬碟
4. [Fail](#fail): 模擬raid 5中一個loop device壞掉
5. [Repair](#repair): 模擬移除壞掉loop device，並加入正常loop device
6. [Remove Raid 5](#remove-raid-5): 完成實驗，解散raid 5、缷載loop devices、並刪除所有disk image

## Create Images
```console
# dd if=/dev/zero of=disk0.img bs=10M count=1
# dd if=/dev/zero of=disk1.img bs=10M count=1
# dd if=/dev/zero of=disk2.img bs=10M count=1
# dd if=/dev/zero of=disk3.img bs=10M count=1
```
## Loop Devices
```console
# sudo losetup -fv disk0.img
# sudo losetup -fv disk1.img
# sudo losetup -fv disk2.img
# sudo losetup -fv disk3.img
```
檢查一下loop devices狀態
```console
# losetup -a
/dev/loop1: []: (/tmp/disk1.img)
/dev/loop2: []: (/tmp/disk2.img)
/dev/loop0: []: (/tmp/disk0.img)
/dev/loop3: []: (/tmp/disk3.img)
```
## Raid 5
```console
# sudo mdadm --create /dev/md0 --auto=yes --level=5 --raid-devices=3 --spare-devices=1 /dev/loop{0,1,2,3}
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```
查看raid裝置狀態，若還沒有完成，會有Rebuild Status來顯示建立raid的進度
```console
# sudo mdadm --detail /dev/md0
/dev/md0:
    Number   Major   Minor   RaidDevice State
       0       7        0        0      active sync   /dev/loop0
       1       7        1        1      active sync   /dev/loop1
       4       7        2        2      active sync   /dev/loop2

       3       7        3        -      spare   /dev/loop3
```
## Fail
假設raid 5當中的/dev/loop0壞掉，原本備援的/dev/loop3會替代壞掉的硬碟，重建raid也會花上一些時間
```console
# sudo mdadm /dev/md0 --fail /dev/loop0
mdadm: set /dev/loop0 faulty in /dev/md0
```
再檢查一下，若還沒有完成，也會有Rebuild Status顯示進度
```console
# sudo mdadm --detail /dev/md0
/dev/md0:
    Number   Major   Minor   RaidDevice State
       3       7        3        0      active sync   /dev/loop3
       1       7        1        1      active sync   /dev/loop1
       4       7        2        2      active sync   /dev/loop2

       0       7        0        -      faulty   /dev/loop0
```
## Repair
```console
# sudo mdadm /dev/md0 --remove /dev/loop0
mdadm: hot removed /dev/loop0 from /dev/md0
# sudo mdadm /dev/md0 --add /dev/loop0
mdadm: added /dev/loop0
# sudo mdadm --detail /dev/md0
/dev/md0:
    Number   Major   Minor   RaidDevice State
       3       7        3        0      active sync   /dev/loop3
       1       7        1        1      active sync   /dev/loop1
       4       7        2        2      active sync   /dev/loop2

       5       7        0        -      spare   /dev/loop0
```
## Remove Raid 5
```console
# sudo mdadm --stop /dev/md0
mdadm: stopped /dev/md0
# sudo losetup -d /dev/loop{0,1,2,3}
# rm disk{0,1,2,3}.img
```
