# `windows`开机自动启动 `wsl` 里的`ssh`
1. 打开`windows`的`task scheduler`
2. `menubar`-> `action`-> `create basic task` ->`when the computer starts` -> `start a program` ->program: `C:\Windows\System32\wsl.exe`, argument: `-d Ubuntu-20.04 -u root service ssh start`
	1. 其中 `-d` 的参数可以用这个命令看 `wsl -l`
3. 在 `Task Scheduler` 里双击那个任务，修改`logon`设定
![[task_scheduler_security_option.png]]
4. 实践之后发现失败了。。。。

