# List ini belum dicoba

## 2022-12-22

- [https://www.windowscentral.com/how-create-task-using-task-scheduler-command-prompt](https://www.windowscentral.com/how-create-task-using-task-scheduler-command-prompt)
- [https://www.digitalcitizen.life/ways-start-task-scheduler-windows/](https://www.digitalcitizen.life/ways-start-task-scheduler-windows/)


### Membuat schedule task via cmd prompt

```
# daily at 11 am
schtasks /create /sc daily /tn "folderpath\taskname" /tr "C:\source\folder\app-or-script" /st 11:00
schtasks /create /sc daily /tn "alpha\notepad" /tr "C:\windows\system32\notepad.exe" /st 11:00

# weekly at sun 11 pm
schtasks /create /sc weekly /d sun /tn "folder\task name" /tr "c:\path\to\script-or-app" /st hh:mm
schtasks /create /sc weekly /d sun /tn "alpha\weekly backup" /tr "C:\backup\weekly.bat" /st 23:00

# monthly task at 11 am
schtasks /create /sc monthly /d 15 /tn "alpha\letsencrypt-renew" /tr "C:\script\letsencrypt-renew.bat" /st 11:00

# daily as spesific user
schtasks /create /sc daily /tn "user\notepad" /tr "C:\users\user\script\runme.bat" /st 11:00 /ru username
schtasks /create /sc daily /tn "admin\backup" /tn "C:\users\admin\script\backup.bat" /st 11:00 /ru admin

# tampilkan semua task
schtasks /query
schtasks /create /?
```

keyword | keterangan
--------|------------------
/create | membuat task scheduled automated
/sc     | opsi: minute, hourly, daily, weekly, monthly, once, onstart, onlogon, onidle, onevent
/d      | day of the week: mon, tue, wed, thu, fri, sat, sun;; utk monthly: 1-31 atau * (wildcard) utk setiap hari
/tn     | task name & location in task library. "task name" vs "path\taskname", lebih rapih utk group task
/tr     | location & script/app name utk dijalankan
/st     | time utk menjalankan task (24h format)
/query  | display semua task
/ru     | jalankan task sbg user tertentu


### Modifikasi Task via cmd

```
# mengubah waktu dijalankan task
schtasks /change /tn "path\taskname" /st hh:mm
schtasks /change /tn "path\taskname" /st 09:00

# mengubah user yg menjalankan task
schtasks /change /tn "path\taskname" /ru username-lain

# disable task
schtasks /change /tn "path\taskname" /disable

# help
schtasks /change /?
```

keyword | keterangan
--------|------------------
/change | edit task
/tn     | task name & location yg akan dimodifikasi
/st     | waktu baru utk menjalankan
/disable| disable task

### Hapus task via cmd

```
# hapus task
schtasks /delete /tn "path\taskname"
schtasks /delete /tn "alpha\weekly"

# help
schtasks /delete /?
```

keyword | keterangan
--------|------------------
/delete | hapus task
/tn     | path & nama task yg akan dihapus

