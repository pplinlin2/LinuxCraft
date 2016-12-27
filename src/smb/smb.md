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
```console
# sudo mount -t cifs //172.17.22.51/data /mnt -o username=qvs
Password for qvs@//172.17.22.51/data:  ****
# df -Th | sed -n -e '1p' -e '/cifs/p'
Filesystem          Type      Size  Used Avail Use% Mounted on
//172.17.22.51/data cifs      450G  5.3G  422G   2% /mnt
```
## Windows client
```console

```
