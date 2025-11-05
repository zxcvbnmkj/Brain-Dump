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
## 工具
### shell 解释器
查看当前使用的解释器
```
echo $SHELL
# 典型输出：
/bin/bash
/bin/zsh
/bin/sh
/bin/tcsh
/bin/csh
```
#### bash：Linux 的默认解释器
- 配置文件在 `~/.bashrc`
#### zsh（Z Shell）: Mac 的默认解释器
- 具有比 bash 更强大的自动补全
- 配置文件：`~/.zshrc`
- `Oh My Zsh` 是一个管理 Zsh 配置的 框架，提供有主题、插件。如：有 git 插件提供简化的 Git 命令
```
gst  # git status
gco  # git checkout
gl   # git pull
```
也可以改变主题
```
# 普通终端
user@hostname ~ %
# 使用 oh my zsh 的流行主题后
➜  ~ git:(main) ✗
```
## 一些常用操作
1. 在日志文件中查找对应的行并输出到终端 `grep "xxx" file.log`

搜索内容的引号可省略
```
[web@ut-gpu bert_check_completed]$ grep "Validation Metrics:" finetuning.log
2025-10-29 17:27:23,918 - INFO - Validation Metrics: 0.8765,  0.893160, 0.708582,0.770833
```
