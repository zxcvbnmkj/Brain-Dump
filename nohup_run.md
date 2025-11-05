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
安装
```
brew install tmux
```

## supervisor
