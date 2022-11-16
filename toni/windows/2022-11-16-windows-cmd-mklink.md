# Membuat Link di windows

> fungsi `mklink` hanya tersedia didalam cmd.exe, tidak didalam PowerShell

```bat
cmd.exe
mklink /?
```

## secara default link file, misal link dari file d:\folder\folder\file1 ke d:\dir\dir\file2

```cmd
d:\> mklink <file-link> <file-target-tujuan>

cd \dir\dir\
mklink file2 d:\folder\folder\file1
rm file2
```

## link folder fold1 ke dir2

```cmd
mklink /d dir2 d:\folder\folder\fold1
rmdir dir2
```

> jangan gunakan `rm` utk menghapus `directory link`, karena akan menghapus semua isi folder tujuannya

# link junction folder

> ini seru nih. jadi dibeberapa bahasa program seperti php, mungkin tidak bisa membaca link biasa/folder
> nah si **junction** ini bisa nih, terakhir gw pake di **phpVirtualBox**

```cmd
mklink /j juncdir2 d:\folder\folder\junction-target
mklink /j d:\contoh-pemendekan-dgjunction d:\folder\aslinya\jauh\didalam\folder\lain
rmdir juncdir
```

![image](https://user-images.githubusercontent.com/89820226/202116248-78335f2f-f919-4253-b9f3-9affe423c986.png)

![image](https://user-images.githubusercontent.com/89820226/202118783-3ed752ed-f7ec-4ae4-8def-897e9a0efe43.png)

source:
- [https://www.addictivetips.com/windows-tips/create-delete-a-junction-link-on-windows-10/](https://www.addictivetips.com/windows-tips/create-delete-a-junction-link-on-windows-10/)
- 
