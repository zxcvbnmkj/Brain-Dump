# 服务器令程序在后台运行方法
通过 ssh 方法连接远程服务器，一定时间没有动作连接就会中断。本机终端连接（scp 对应的接法）、finalshell 等软件连接都会有这个问题
## nohup
通过在运行命令前面加上 nohup 来挂载程序进程到后台，使得程序与终端没有关系

在末尾加上 & 可以使得执行完命令后立刻关闭这次对话，不加的话也可以使用 ctrl + C 手动结束，这不会影响程序执行。
```
$> nohup python main.py &
[1] 367592
nohup: 忽略输入并把输出追加到 'nohup.out'
```
指定控制台内容输出文件：`>` 表示重定向；`&1` 表示标准输出当前的位置
> - `0`: 标准输入
> - `1`: 标准输出
> - `2`: 错误输出

- 只重定向标准输出，错误输出仍到 nohup.out
```
nohup python main.py > custom.out &
```
- 标准输出和错误输出到不同文件
```
nohup python main.py > custom.out 2> custom.err &
```
- 标准输出和错误输出放到同一个文件
```
nohup python main.py > custom.out 2>&1 &
```
查看使用 nohup 挂载的进程情况（**仅在终端未断开过连接时有效**）
```
$> jobs
[1]+  运行中               nohup python train_512_char.py &
```
`jobs` 失效后找到该进程编号
```
ps aux | grep main.py
```
知道编号后查看该进程的运行状态和持续运行时间
```
$ ps -p 367592
PID TTY          TIME CMD
367592 pts/0    00:26:50 python
```
终止挂载的进程
```
kill <PID>
```
## tumx
一般我们打开一个**终端窗口**可以看做是建立了一个**会话**，关闭**窗口**时，**会话**也断开，而 `tumx` 可让窗口和会话解绑。窗口关闭但会话依旧保持。

**需要先连接上远程主机，再到远程主机的终端里面执行 tmux，而不是本地主机 tmux 再连远程**
- 安装
```
brew install tmux
```
- 启动：打开新窗口后输入 `tmux`。左下角显示 tmux 的编号，第一个启动的 tmux 窗口，编号是 0，第二个窗口的编号是 1 ......
- 使用数字命名不好记忆，可以在启动 tmux 的时候指定会话名字
```
tmux new -s <session-name>
```
- 退出当前 Tmux 窗口，但是会话和里面的进程仍然在后台运
```
tmux detach
```
- 查看本机所有的 tmux 会话
```
输入：tmux ls
0: 1 windows (created Wed Nov  5 09:44:45 2025) (attached)
```
- 重新接入一个会话
```
tmux attach -t 0    //使用会话编号
tmux attach -t <session-name>   使用会话名称
```
- 杀死会话
```
tmux kill-session -t 0               //使用会话编号
tmux kill-session -t <session-name>  //使用会话名称
```
- 切换会话
```
tmux switch -t 0                   // 使用会话编号
tmux switch -t <session-name>      // 使用会话名称
```
- 退出：输入 `exit` 或按 `Ctrl + D`
## supervisor
