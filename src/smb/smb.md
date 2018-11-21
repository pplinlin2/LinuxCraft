# SMB
## Flow
1. [Server setting](#server-setting): 安裝SMB server，並設定相關config檔
1. [Linux client](#linux-client): 從linux掛載SMB資料夾
1. [Windows client](#windows-client): 從windows掛載SMB資料夾

## Server setting
下載SMB server相關的套件
```console
# sudo apt-get install samba
```
將/data加入samba server，設定要須要帳密登入，執行testparm檢查config檔
```console
# tail /etc/samba/smb.conf
[global]
    follow symlinks = yes
    unix extensions = no
    wide links = yes
    # https://access.redhat.com/solutions/54407
[data]
    path = /data
    browsable = yes
    guest ok = no
    writable = yes
# testparm
Load smb config files from /etc/samba/smb.conf
Processing section "[printers]"
Processing section "[print$]"
Processing section "[data]"
Loaded services file OK.
```
加入user、並檢查是否加入成功，注意user的帳號必須本來就是linux user
```console
# sudo pdbedit -a qvs
new password: ****
retype new password: ****
# sudo pdbedit -L
qvs:1000:qvs
```
## Linux client
```console
# sudo apt-get install smbclient cifs-utils
```
使用smbclient來連接smb server，使用起來很像ftp server
```console
# smbclient //172.17.22.51/data -U qvs
WARNING: The "syslog" option is deprecated
Enter qvs's password:
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.4.5-Ubuntu]
smb: \> ls
  .                                   D        0  Wed Dec 28 09:51:51 2016
  ..                                  D        0  Tue Dec 27 18:52:45 2016
  img                                 D        0  Wed Dec 28 09:51:04 2016
  index.html                          N        0  Wed Dec 28 09:51:51 2016

                471623912 blocks of size 1024. 442097024 blocks available
smb: \> get index.html
getting file \index.html of size 0 as index.html (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \> cd img
smb: \img\> put header.jpg
putting file header.jpg as \img\header.jpg (0.0 kb/s) (average 0.0 kb/s)
smb: \img\> ls
  .                                   D        0  Wed Dec 28 09:53:41 2016
  ..                                  D        0  Wed Dec 28 09:51:51 2016
  header.jpg                          A        0  Wed Dec 28 09:53:41 2016

                471623912 blocks of size 1024. 442097024 blocks available
smb: \img\> quit
```
用mount將smb資料夾掛載起來，使用get/put來上下載檔案
```console
# sudo mount -t cifs //172.17.22.51/data /mnt -o username=qvs
Password for qvs@//172.17.22.51/data:  ****
# df -Th | sed -n -e '1p' -e '/cifs/p'
Filesystem          Type      Size  Used Avail Use% Mounted on
//172.17.22.51/data cifs      450G  5.3G  422G   2% /mnt
```
## Windows client
```console
> net view \\172.17.22.51
共用資源在 \\172.17.22.51
qvs-ThinkCentre-M92p server (Samba, Ubuntu)

共用名稱  類型  使用方式  註解
-------------------------------------------------------------------------------
data      Disk
命令已經成功完成。

> start \\172.17.22.51
```

<kbd><image src="https://github.com/pplinlin2/LinuxCraft/blob/master/src/smb/connect.png" width="500"/></kbd>
