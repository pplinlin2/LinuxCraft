# iSCSI
## Flow
1. [Server setting](#server-setting): 在Linux上用一個disk image來架設iSCSI target
1. [Linux client](#linux-client): 使用Linux當做iSCSI initiator連接iSCSI target
1. [Windows client](#windows-client): 使用Windows當做iSCSI initiator連接iSCSI target

## Server setting
安裝iSCSI相關套件
```console
# sudo apt-get install tgt
```
建立一個空的disk image來當作LUN，並修改設定檔
```console
# dd if=/dev/zero of=/tmp/disk.img bs=100M count=1
# tail /etc/tgt/targets.conf
<target iqn.2016-12.com.example:emudisk>
        backing-store /tmp/disk.img
        incominguser qvs 012345678910
</target>
```
啟動tgt服務，可以從tgtadm觀察tgt狀態，或是監聽port 3260看iSCSI server有沒有啟動
```console
# sudo service tgt start
# sudo tgtadm --mode target --op show
Target 1: iqn.2016-12.com.example:emudisk
    System information:
        Driver: iscsi
        State: ready
    I_T nexus information:
    LUN information:
        LUN: 0
            Type: controller
            SCSI ID: IET     00010000
            SCSI SN: beaf10
            Size: 0 MB, Block size: 1
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: null
            Backing store path: None
            Backing store flags:
        LUN: 1
            Type: disk
            SCSI ID: IET     00010001
            SCSI SN: beaf11
            Size: 105 MB, Block size: 512
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: rdwr
            Backing store path: /tmp/disk.img
            Backing store flags:
    Account information:
        qvs
    ACL information:
        ALL
# sudo netstat -tlunp | grep 3260
tcp        0      0 0.0.0.0:3260            0.0.0.0:*               LISTEN      26819/tgtd
tcp6       0      0 :::3260                 :::*                    LISTEN      26819/tgtd
```
如果已經有iSCSI initiator已經連接，也可以用`sudo tgtadm --mode target --op show`查看
```console
# sudo tgtadm --mode target --op show
Target 1: iqn.2016-12.com.example:emudisk
    System information:
        Driver: iscsi
        State: ready
    I_T nexus information:
        I_T nexus: 4
            Initiator: iqn.1993-08.org.debian:01:516b7dd19514 alias: ubuntu
            Connection: 0
                IP Address: 172.17.28.138
...
```
## Linux client
安裝iSCSI initiator相關套件
```console
# sudo apt install open-iscsi
```
修改設定檔，啟動iSCSI initiator
```console
# sudo vim /etc/iscsi/iscsid.conf
node.session.auth.authmethod = CHAP
node.session.auth.username = qvs
node.session.auth.password = 012345678910
discovery.sendtargets.auth.authmethod = CHAP
discovery.sendtargets.auth.username = qvs
discovery.sendtargets.auth.password = 012345678910
# sudo service iscsid start
```
查看iSCSI target的資訊，並登入
```console
# sudo iscsiadm --mode discovery --type sendtargets --portal 172.17.22.51
172.17.22.51:3260,1 iqn.2016-12.com.example:emudisk
# sudo iscsiadm --mode node --targetname iqn.2016-12.com.example:emudisk --portal 172.17.22.51 --login
Logging in to [iface: default, target: iqn.2016-12.com.example:emudisk, portal: 172.17.22.51,3260] (multiple)
Login to [iface: default, target: iqn.2016-12.com.example:emudisk, portal: 172.17.22.51,3260] successful.
```
從dmesg log可以看到/dev/sda被掛載起來，用fdisk檢查果真是我們剛才掛載的100M iSCSI target
```console
# dmesg | tail
[4856940.539724] scsi host2: iSCSI Initiator over TCP/IP
[4856941.553808] scsi 2:0:0:0: RAID              IET      Controller       0001 PQ: 0 ANSI: 5
[4856941.558784] scsi 2:0:0:0: Attached scsi generic sg1 type 12
[4856941.563073] scsi 2:0:0:1: Direct-Access     IET      VIRTUAL-DISK     0001 PQ: 0 ANSI: 5
[4856941.565513] sd 2:0:0:1: Attached scsi generic sg2 type 0
[4856941.567999] sd 2:0:0:1: [sda] 204800 512-byte logical blocks: (105 MB/100 MiB)
[4856941.568002] sd 2:0:0:1: [sda] 4096-byte physical blocks
[4856941.570849] sd 2:0:0:1: [sda] Write Protect is off
[4856941.570852] sd 2:0:0:1: [sda] Mode Sense: 69 00 10 08
[4856941.572282] sd 2:0:0:1: [sda] Write cache: enabled, read cache: enabled, supports DPO and FUA
[4856941.581759]  sda:
[4856941.589181] sd 2:0:0:1: [sda] Attached SCSI disk
# sudo fdisk -l /dev/sda
Disk /dev/sda: 100 MiB, 104857600 bytes, 204800 sectors
```
卸載iSCSI target
```console
# sudo iscsiadm --mode node --targetname iqn.2016-12.com.example:emudisk --logout
Logging out of session [sid: 1, target: iqn.2016-12.com.example:emudisk, portal: 172.17.22.51,3260]
Logout of [sid: 1, target: iqn.2016-12.com.example:emudisk, portal: 172.17.22.51,3260] successful.
# sudo iscsiadm --mode node --targetname iqn.2016-12.com.example:emudisk -o delete
```
## Windows client
在命令提示字元執行，
```console
> iscsicpl.exe
```
<img src="https://github.com/pplinlin2/LinuxCraft/blob/master/src/iscsi/step1.png" width="600"/>

因為我們有設定CHAP帳密設定，所以要點選進階，啟動CHAP登入

<kbd><img src="https://github.com/pplinlin2/LinuxCraft/blob/master/src/iscsi/step2.png" width="300"/></kbd>
<img src="https://github.com/pplinlin2/LinuxCraft/blob/master/src/iscsi/step3.png" width="600"/>

設定成功後，就可以從磁碟管理看到我們剛剛連線的100M磁碟了

<kbd><img src="https://github.com/pplinlin2/LinuxCraft/blob/master/src/iscsi/step4.png" width="400"/></kbd>
