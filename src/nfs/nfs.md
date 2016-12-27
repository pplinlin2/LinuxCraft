# NFS
## Flow
1. [Server setting](#server-setting): 在Linux上NFS server的架設及相關的設定
1. [Linux client](#linux-client): 用Linux掛載NFS server目錄
1. [Windows client](#windows-client): 用Windows掛載NFS server目錄

## Server setting
下載NFS server相關的套件
```console
# sudo apt-get install nfs-kernel-server
```
建立要分享的資料夾，並修改NFS的設定檔
```console
# sudo mkdir /data
# sudo chmod -R 777 /data
# cat /etc/exports
/data *(rw,sync,root_squash,no_subtree_check)
# sudo exportfs -rv
exporting *:/data
# showmount -e localhost
Export list for localhost:
/data *
```

## Linux client
檢查server ip上所有的掛載點
```console
# NFS=172.17.22.51
# showmount -e ${NFS}
Export list for 172.17.22.51:
/data *
```
掛載到/mnt上，成功後就可以進行資料讀寫了！
```console
# sudo mount -t nfs ${NFS}:/data /mnt
# mount | grep nfs
172.17.22.51:/data on /mnt type nfs4 (rw,relatime,vers=4.0,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=172.17.28.138,local_lock=none,addr=172.17.22.51)
```
卸載NFS
```console
# sudo umount /mnt
```
## Windows client
開啟Windows的NFS client功能

<kbd><img src="https://github.com/pplinlin2/LinuxCraft/blob/master/src/nfs/setting.png" width="600"></kbd>

```console
> showmount -e 172.17.22.51
匯出清單 172.17.22.51:
/data                              *

> mount \\172.17.22.51\data *
Z: 現在已成功連接到 \\172.17.22.51\data
命令已經成功完成。
```
掛載完成後，可以看到Z槽的出現

<kbd><img src="https://github.com/pplinlin2/LinuxCraft/blob/master/src/nfs/success.png" width="600"></kbd>

卸載Z槽
```console
> umount Z:
正在中斷連線            Z:      \\172.17.22.51\data
命令已經成功完成。
```
