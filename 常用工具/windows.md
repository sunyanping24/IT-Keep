# windows常用命令
```
netstat -aon  // 查找所有端口
netstat -aon | findstr "8080" // 查找8080端口的程序
tasklist | findstr "PID"  // 列出进程号对应的所有子进程
taskkill /f /t /im 任务名称  // 杀掉执行的任务
```
