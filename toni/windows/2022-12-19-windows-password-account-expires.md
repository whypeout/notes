sources:
- [https://www.shellhacks.com/password-never-expires-disable-password-expiration/](https://www.shellhacks.com/password-never-expires-disable-password-expiration/)
- [https://www.windows-commandline.com/set-password-to-never-expire/](https://www.windows-commandline.com/set-password-to-never-expire/)
- 
# Disable Password Expiration di Windows

menggunakan "Local Users and Groups" [Windows + R](#) `lusrmgr.msc` -> buka sub "Users" -> klik kanan <nama-user> -> Properties -> Ceklis [v] Password Never Expires
  
menggunakan Powershell sbg Administrator, search `powershell` Ctrl+Shift+Enter utk run as administrator
  
```
PS C:\>
  Get-LocalUser | Where-Object { $_.Enabled } | Select Name,PasswordExpires
  
  Set-LocalUser -Name "Username" -PasswordExpires 1
```

menggunakan cmd `wmic`, buka cmd sbg administrator
  
```
net user
net user it-dept
wmic UserAccount Where "Name='username'" Set PasswordExpires=FALSE
net user it-dept | findstr /c:"Password Expires"
```
