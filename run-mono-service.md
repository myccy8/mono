1.安装 Supervisor
```Bash
# yum install supervisor
```
2.新建`supervisor.log` 目录
```
$ mkdir ~/projects/supervisor.log
```
3.新建项目的配置文件
```Bash
$ echo_supervisord_conf >  ~/projects/supervisord.conf
$ vim ~/projects/supervisord.conf
```
```supervisord
[program:mobileapi]
command=mono /apps/home/mintcode/projects/Mintcode.Dianna/SourceCode/Mintcode.Dianna.Chat/Mintcode.Dianna.MobileApi-Mono/bin/Debug/Mintcode.Dianna.MobileApi-Mono.exe
process_name = mobileapi
autorstart=true
stdout_logfile=~/projects/supervisor.log/mobileapi.log

[program:serverBus]
command= mono /apps/home/mintcode/projects/Mintcode.Dianna/SourceCode/Mintcode.Dianna.Chat/Service\ Bus/Mintcode.Dianna.ServiceBus/bin/Debug/Mintcode.Dianna.ServiceBus.exe
process_name = serverBus
autorstart=true
stdout_logfile=~/projects/supervisor.log/serverBus.log

[program:ws]
command=mono /apps/home/mintcode/projects/Mintcode.Dianna/SourceCode/Mintcode.Dianna.Chat/WebSocket/Mintcode.Dianna.WindowsService.WebSocket/bin/Debug/Mintcode.Dianna.WindowsService.WebSocket.exe
process_name = ws
autorstart=true
stdout_logfile=~/projects/supervisor.log/ws.log

[program:chat]
command=fastcgi-mono-server4 --applications=chat.sirius:/:/apps/home/mintcode/projects/Mintcode.Dianna/SourceCode/Mintcode.Dianna.Chat/Mintcode.Dianna.Chat --socket="tcp:127.0.0.1:10001" --printlog
process_name = chat
autorstart=true
stdout_logfile=~/projects/supervisor.log/chat.log
```
4.运行命令
```Bash
$ supervisord -c ~/projects/supervisord.conf
```
5.测试命令
```Bash
$ supervisord -c ~/projects/supervisord.conf -n
```
6.进入管理命令
```Bash
$ supervisorctl  -c ~/projects/supervisord.conf
```