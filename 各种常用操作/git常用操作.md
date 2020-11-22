# Ubuntu 上装 git server
0. 参考资料 https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server
1. 确保ubuntu上有 ssh 运行
	1. `sudo apt install openssh-server`
	2. `sudo service ssh start`
2. 创建一个 repo 的专用 Ubuntu user
	1. `sudo adduser myuser`
3. 在该 user 里为允许的 client 机器配置 ssh 权限
	1. `su myuser`
	2. `mkdir ~/.ssh`
	3. `chmod 700 ~/.ssh`
	4. `touch ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys`
	5. `vim ~/.ssh/authorized_keys`，添加目标client 的 ssh pub key
4. 创建 git 项目
	1. `mkdir ~/xxx.git && cd ~/xxx.git`
	2. `git init --bare`
5. 这个git项目的地址是 `myuser@ip:/~/xxx.git`
6. 为了安全，还可以在 ~/.ssh/authorized_keys 里限制**每一个** client 的行为
```
no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty
公钥1
```

# Windows 上 git CRLF
0. 参考资料
1. Unix 上换行只有 LF，Windows 上是 CRLF
	1. 如果一个文件现在 Unix 上编辑后在 Windows 上编辑，git 会报 CRLF 改动。反之亦然。
2. `git autocrlf` 可以自动帮助修改文件（而**不是隐藏**改动）
```
core.autocrlf=true:      core.autocrlf=input:     core.autocrlf=false:
                                             
        repo                     repo                     repo
      ^      V                 ^      V                 ^      V
     /        \               /        \               /        \
crlf->lf    lf->crlf     crlf->lf       \             /          \      
   /            \           /            \           /            \
```
所以可以看到
```
1) true:    x -> LF -> CRLF   warning "LF will be replaced by CRLF"
2) input:   x -> LF -> LF     warning "CRLF will be replaced by LF"
3) false:   x -> x -> x       no warning
```
3. 所以如果要做 Unix 上用，就用 =true 最好。

# Git status 文件路径中unicode显示错误
错误类似 ` modified: "\345\220/\234.md"`
这是因为 git 默认对 非ascii 的路径字符都 quote。
可以这样解决 `git config --global core.quotepath false`

# git track 到已有branch
`git branch -u origin/master`