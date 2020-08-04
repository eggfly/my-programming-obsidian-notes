# 只传第二个参数
假设脚本里这么写:
``` powershell
Param(
  [string]$a = "aaaa",
  [string]$b = "bbbb",
  [string]$c = "cccc",
  [string]$d = "dddd"
)
```
运行时候只想改第二个，则可以
```powershell
powershell foo.ps1 -b xxxxx
```