# 知源配置

1. 查看本机SSH后，将SSH公钥复制到知源中。
```
# MAC
`cat ~/.ssh/id_rsa.pub`
# Windows 的 cmd 中
type "%USERPROFILE%\.ssh\id_rsa.pub"
```
2. 把知源中的笔记克隆到本地
```
git clone ssh://git@zhiyuanbiji.cn:29492/zhangli2025/essay.git
```
3. 使用 `git` 提交命令，把本地文件 `push` 到知源
