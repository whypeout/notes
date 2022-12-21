# Windows Command Line

sources:
- [https://www.windows-commandline.com/tasklist-command/](https://www.windows-commandline.com/tasklist-command/)
- [https://www.windows-commandline.com/taskkill-kill-process/](https://www.windows-commandline.com/taskkill-kill-process/)
- 

## TaskList command

```
# list all running process
tasklist

# list process using memory greater than KB, e.g 30MB == 30000 KB
tasklist /fi "memusage gt memorysize"
tasklist /fi "memusage gt 30000"

# list process launch by user
tasklist /fi "username eq administrator"

# find memory usage of spesific process
tasklist /fi "pid eq processID"
tasklist /fi "pid eq 6544"

# find not responding process
tasklist /fi "status eq not responding"

# list services running in process
tasklist /svc /fi "pid eq processID"
tasklist /svc /fi "pid eq 624"

# list process running uptime
tasklist /fi "cputime gt hh:mm:ss"
tasklist /fi "cputime gt 01:20:00"

# find process image file
tasklist /fi "imagename eq imageName"
tasklist /fi "imagename eq VBoxWebSrv.exe"
tasklist /fi "imagename eq firefox.exe"

# find process running from service
tasklist /fi "services eq serviceName"
tasklist /fi "services eq webclient"
```

## TaskKill command

```
# kill berdasarkan nama file
taskkill /IM filename.exe
taskkill /IM vboxgui.exe

# kill app force
taskkill /IM /F iexplore.exe
taskkill /IM /F explorer.exe

# kill process id
taskkill /PID 1234

# kill process high mem in KB. 100000 = 100MB
taskkill /FI "memusage gt KB"
taskkill /FI "memusage gt 100000"


```


