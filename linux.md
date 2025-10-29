# Linux
## 一、连接服务器
（一）直接通过终端
```
ssh ubuntu@local.nowcoder.com -p 50023
```
## 二、数据的上传与下载- - -通过`scp`
在**本地电脑(上传与下载都是)** 中执行以下命令，其中`-r`表示上传的是文件夹，不加参数则默认为文件。

如果端口是默认的`22`可以省略，否则必须要有大写的 P 指定端口号。

传递的参数 1 表示文件夹在本地的路径；参数 2 为远程服务器的路径，写作`用户名@主机IP:路径`
```
scp -P 50023 -r /private/tmp/yes.xlsx ubuntu@local.nowcoder.com:~/
```
若在上传时需要排除某些的文件
```
rsync -avz -e ssh \
--exclude='__pycache__' \
--exclude='*.log' \
--exclude='*.pyc' \
--exclude='bert_classifier/' \
/Users/nowcoder/workspace/bert_classification/ 用户名ß@服务器IP:~/workspace/bert_classification/
```
把源文件夹和目标文件夹的顺序反一下就是“从服务器下载文件夹”的命令，但是要注意的是，源路径需要指定需上传的文件夹层，目标路径则只需指定到上一层即可。
```
scp -P 50023 -r ubuntu@local.nowcoder.com:~/files /private/tmp/
```
## 三、问题解决
1. finalshell 等软件若检测客户端很长时间没有动作则会自动断开连接，若在训练模型且是命令行直接训练的，没有挂载，则训练会中断掉。可以参照[以下帖子](https://blog.csdn.net/weixin_69218754/article/details/132402412)的做法，去更改服务器 SSH 的配置文件，使得每隔一段时间自动发送信息。
2. 建议使用 nohug python xx.py 方法，这样即使终端断开也依旧不影响进程
