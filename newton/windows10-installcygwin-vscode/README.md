[how to install cygwin](https://www3.ntu.edu.sg/home/ehchua/programming/howto/Cygwin_HowTo.html)

1.download cygwin setup here
2.setup cygwin pertama kali (internet), pilih folder download yg diinginkan
misal: install to c:\cygwin64; dan local directory c:\cygwin_package
pilih source/repository server: nus.edu.sg
3.pilih packet yg diinginkan: openssh, openssl, tmux, coreutils, curl, bash, zsh, git, rsync
4.tambahkan PATH environment variable c:\cygwin64
Control Panel -> System and Security -> System Advanced System Settings -> Advanced tab -> Environment Variables -> System Variables -> Select variable PATH in system -> Edit -> Add c:\cygwin64 -> Apply/OK/Close
5.buka cmd.exe
```
mkpasswd -l > /etc/passwd
mkgroup -l > /etc/group
```
