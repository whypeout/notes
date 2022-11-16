# VirtualBox Web Service

vboxwebsrv

export VBOXWEB_$PREFIX

- USER utk menjalankan web service. windows bisa dg spesifik "administrator" atau "system local account"
- HOST default 127.0.0.1, pakai 0.0.0.0 utk multiple server config
- PORT default 18083
- SSL_KEYFILE server_key dan certificate_file, format PEM
- SSL_PASSWORDFILE filename for password to server_key
- SSL_CACERT ca cert file, format PEM
- SSL_CAPATH ca cert path
- SSL_DHFILE dh file/dh key length in bits
- SSL_RANDFILE random number generator
- TIMEOUT 300 session timeout in seconds, 0 disables timouts
- CHECK_INTERVAL default 5. freq timeout in secs
- THREADS default 100 max worker threads in paralel
- KEEPALIVE default 100 max requests before a socket closed
- ROTATE 10, log file 0 disable
- LOGSIZE 1MB max log size to rotate, bytes
- LOGINTERVAL 1 day, max interval log rotation

vboxwebsrv.exe --host 0.0.0.0 --port 18083 --pidfile c:\users\public\vbox.pid \
--logfile c:\users\public\vbox.log

source:
- [https://docs.oracle.com/en/virtualization/virtualbox/6.0/admin/vboxwebsrv-daemon.html](https://docs.oracle.com/en/virtualization/virtualbox/6.0/admin/vboxwebsrv-daemon.html)

