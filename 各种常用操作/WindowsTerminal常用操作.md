# 打开各种终端
![[windows_terminal_new_shell.png]]

# 添加终端
在设置里的 profiler list 里添加这个：
```
            {
                "guid": "{b453ae62-4e3d-5e58-b989-0a998ec441b9}",
                "name": "Git Bash",
                "commandline": "\"%PROGRAMFILES%\\git\\bin\\bash.exe\" --login -i -l",
                "closeOnExit": true,
                // "historySize": 9001,
            }
```
![[windows_terminal_add_shell.png]]
![[windows_terminal_setting_profiler_add_shell.png]]