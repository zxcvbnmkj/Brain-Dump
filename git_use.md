# Git

## 一、使用之前
设置本台设备的全局用户名与邮箱，在提交代码时即可显示本次提交者的信息
```
# win 系统需要先安装 Git Bash
git config --global user.name zhangli
git config --global user.email 2693175110@qq.com
```
## 二、常用命令
- 查看 ”工作区“ 与 ”暂存区“ 的状态
```
git status
```
- 查看工作区与暂存区的差异

```
git diff
```
- 查看本项目的所有分支
```
git branch
```
- 查看远程分支的信息，重要！！
```
git remote -v
```
-  删除远程仓库
```
git remote remove origin
```
- 储藏当前工作区更改
```
git stash
```
-  恢复 stash 到工作区
```
git stash pop
```
## 三、分支命名
1. 远程仓库的默认别名一般是 `origin` 。我们可以在连接远程仓库的时候，自己指定仓库别名是什么。
```
git remote add origin(此处为远程仓库别名，也可为 upstream 等) https://github.com/zxcvbnmkj/Brain-Dump.git
```
2. 本地仓库的默认主分支名字是 `master`，但 github 中一般认为主分支为 `main` 。若在 github 新建一个空仓库，则会有引导教程：

```
git remote add origin https://github.com/zxcvbnmkj/Brain-Dump.git
# 意为 将本地仓库的 main 分支推送到 origin 仓库的 同名分支
git push -u origin main
```
若我们直接复制第 2 行的代码，会遇到报错（如下），原因是本地没有叫做 `main` 的分支，因此我们需要把该句中的 `main` 改为 `master`
```
error: src refspec main does not match any
error: failed to push some refs to 'https://github.com/zxcvbnmkj/Brain-Dump.git'
```
> **[Note：]**
>
> `-u` 是 `--set-upstream` 的简写，意为设置上游分支。第一次 push 时需要带上。它设置本地的 `master` 分支追踪远程的 `origin/master` 分支。若不加它，以后的每一次 push 都需要写明推送到远程仓库的哪个分支。
3. 一般使用 `origin/main` 来指代与本地仓库当前分支相连的远程仓库分支，其格式是 `仓库名/分支名` 。
## 四、从远程仓库克隆代码
### （一）从 github 下载代码（可省略解压缩这一步）
1. 不需要把本机公钥配置到  github 账号即可直接克隆。
> **[Note : ]**
>
> 只读操作不需要配置，只有写入需要配置 SSH


2. 复制 github  对应仓库的 `.git` 链接，然后 `cd` 到想要保存该项目的路径后，终端执行：
```
git clone https://github.com/sfeng18/microtubule.git
```
3. 问题解决- - -最常见的两个问题

(1) 该问题是因为开启了 VPN ，需要关闭
```
fatal: unable to access 'https://github.com/vexing-shusher/microtubule-dynamics-simulation.git/': LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443
```
(2) 由于“网络原因”
```
E:\gits>git clone https://github.com/vexing-shusher/microtubule-dynamics-simulation.git
Cloning into 'microtubule-dynamics-simulation'...
error: RPC failed; curl 28 Recv failure: Connection was reset
fatal: expected flush after ref listing
```
可通过 “增加 Git 的传输缓冲区大小” 解决，输入：
```
git config --global http.postBuffer 524288000
```
### （二）若远程仓库有多个分支

- 执行正常的克隆操作，完毕后查看下载的文件，可发现只有默认分支的代码。
- 但其实所有分支的代码都被一同克隆下来了，只是默认本地文件夹不显示其余分支
- `cd` 到克隆的文件夹里，查看它有哪些分支

```
E:\AgroSpec\AgroSpec-Vue>git branch -a
* main    # 星号指示当前所在分支
  remotes/origin/HEAD -> origin/main   #远程仓库默认分支
  remotes/origin/main
  remotes/origin/master
```

- 再切换进所需分支中

```
E:\AgroSpec\AgroSpec-Vue>git checkout master
Switched to a new branch 'master'
branch 'master' set up to track 'origin/master'.
```

- 如果不需要某个分支，可以把它删除（需保证当前没有切入进该分支）。如果无法删除，可以使用参数 `-D` 强制删除

```
git branch -d main
```

## 五、从头开始：利用 git 管理项目并提交到远程仓库

### (一) 新建 `.gitignore` 文件

最好在初始化 git 之前创建它，它的默认值可以设置为：

- 对于 python 项目

```
__pycache__/
.ipynb_checkpoints/
.DS_Store
.idea/
*.log
#若要保留某个文件夹，但忽略其内部所有文件
data/*
```

- 对于 Java 项目

```
.idea
*.iml
```

### (二) 初始化 Git 仓库

初始化完毕后会在项目根目录下新建一个名为 `.git`  的**隐藏文件**，并且编译器（pycharm、VSCode 等）中终端的命令行开头会新增一个`master`标志，这说明当前分支是主分支（主线）。

```
$ git init
Initialized empty Git repository in /Users/nowcoder/workspace/david_backend_local/.git/
```
可使用 `ls -la` 查看隐藏文件
![](https://pic-gino-prod.oss-cn-qingdao.aliyuncs.com/zhangli2025/20250926105838674-paste.png)

> **[ Tips : ]**
>
> 建议等自己的项目代码写的差不多了之后，此时丢失代码会有较大损失时，再进行版本控制。没必要对控制追踪新项目，这样会会显得非常乱。

### (三) 查看当前工程各文件状态

该命令会显示本项目中各个文件的状态，非常方便我们观察接下来要对哪些文件进行版本控制

- 哪些文件正在被追踪（当然刚初始化 git 的时候不存在这种文件）
- 哪些文件未被追踪
- 哪些被追踪的文件发生了修改（刚初始化时也不存在这种文件）
- 哪些被追踪的文件被删除了（刚初始化时不存在这类）

```
git status
```

### (四)  添加文件进暂存区

- 将一个文件添加进暂存区
```
git add ./aaa.py
```
- 将多个文件放入暂存区
```
git add ./aaa.py ./bbb.py ./ccc.json
```
- 将本项目中，除被 `.gitignore` 指定忽略，以外的所有文件全部添加进暂存区
```
git add -A
```

### (五) 提交暂存区内容，建立永久快照，通过日志记录本次变动

`-m` 是 Messege 的缩写，引号中是本次提交内容的简短描述。完成后可通过 `git log` 或者编译器的 git 工作区查看提交记录。

```
git commit -m 'init'
```

### (六) 连接远程仓库

```
git remote add origin https://github.com/zxcvbnmkj/Brain-Dump.git
```

### (七) 推送本次 commit 的内容到远程仓库

- 完整形式

```
git push -u origin <本地分支名>:<远程分支名>
```
- 简写形式，可省略远程分支的名字，此时会默认其与本地分支名字一样。第一次推送时一定要指定参数 `-u` ，原因见第三节。
```
git push -u origin master
```
### (八)  设备认证

若当前设备从未连接过自己的 github 账号，则第七步将显示需要登陆 github，但是如果只是按照提示输入账号密码，则会认证失败。这是由于 github 从2021年8月13日开始不再接受账户密码来验证 Git 操作，需要使用 Personal Access Token（个人访问令牌）来代替密码。

```
(sam2) ~/桌面/SAM2微调 git:[master]
git push -u origin master
Username for 'https://github.com': zxcvbnmkj
Password for 'https://zxcvbnmkj@github.com': 	【密码已输入，只是命令行默认隐藏】
remote: Invalid username or token. Password authentication is not supported for Git operations.
fatal: 'https://github.com/zxcvbnmkj/MT.git/' 鉴权失败
```

登陆 github 网页后，`Settings → Developer settings(位于左侧栏最下方)`，然后选择 `Tokens (classic)`，即可生成一个令牌，它可代替密码。需注意，该令牌的内容只在第一次生成时可见，最好妥善保存一下。

```
(sam2) ~/桌面/SAM2微调 git:[master]
git push -u origin master
Username for 'https://github.com': zxcvbnmkj
Password for 'https://zxcvbnmkj@github.com':
枚举对象中: 41, 完成.
对象计数中: 100% (41/41), 完成.
使用 80 个线程进行压缩
压缩对象中: 100% (38/38), 完成.
写入对象中: 100% (41/41), 15.69 MiB | 8.24 MiB/s, 完成.
总共 41（差异 0），复用 0（差异 0），包复用 0
To https://github.com/zxcvbnmkj/MT.git
 * [new branch]      master -> master
分支 'master' 设置为跟踪来自 'origin' 的远程分支 'master'。
```

## 六、将远程仓库的更新同步到本地

**！！！ 拉取远程更新不建议使用 `git pull`  ！！！  应当采用以下方法：**

- 第一步，获取远程仓库最新提交日志
```
git fetch
```
- 第二步，将本地提交置于远程仓库最新提交的基础上，从而保持线性日志，避免分叉
```
git rebase
```
- **与上述两步操作等价的简洁写法**
```
git pull --rebase
```
> **[ Note :  ”  关于为什么不建议使用 `pull`  “]**
>
> 拉取远程更新，最简单的办法就是 `git pull`，但是这样会导致远程仓库提交线产生分叉。若远程仓库和本地仓库的提交线为：
>
> ```
> 远程：A - B - C (同学推送 C )
> 本地：A - B - D (本地还未 pull 的新版本)
> ```
>
> `pull` 操作会自动把远程更新与本地代码合并，得到一个融合版本 M
>
> ```
> git pull` = `git fetch` + `git merge
> ```
> 若此时执行 push ，将把 M 推送到远程仓库，则远程仓库提交线为：
> ```
> A - B - C - M (合并提交)
>     ↘  ↗
>       D (本地更新)
> ```
> 因此，`pull` 的不足之处主要在于，它会自动 `git merge`
> 为避免上述情况，我们需要采用 `git rebase` 来代替 `git merge`，它们两种命令都会自动把远程更新同步到本地，只是前者比后者多了一个将本次提交置于远程仓库最新提交之后的功能。这样就可得到以下提交线：
> ```
> A - B - C - D
> ```

## 七、有新更改时将其同步到远程仓库

### (一) 查看文件中增删改查的部分

在 pycharm 中按下快捷键 `ctrl + K`，即可查看。若有不必要的变动，点击回退即可。

### (二) 追踪并提交的新文件

- 添加部分文件，或有新建文件需要纳入追踪
    ```
    git add ./aaa.py
    git commit -m 'md'
    ```
###  (三) 提交发生变动的已追踪文件

- 若我 `add` 了一些新文件，与此同时先前已被 `add`（纳入追踪）的旧文件也发送了变动，此时进行 `commit` 只会提交新文件。输入 `git status` 会得到提示，有些文件发送了变动，可以把它们进行 `add` 处理，或者使用 `restore` 丢弃这种变动。

  ```
  尚未暂存以备提交的变更：
    （使用 "git add <文件>..." 更新要提交的内容）
    （使用 "git restore <文件>..." 丢弃工作区的改动）
  	修改：     .gitignore
  	修改：     README.md
  	修改：     premodel/placeholder.txt
  	修改：     "\351\270\241\350\202\213\346\226\207\344\273\266/readme.txt"
  ```

  此时可以使用以下命令自动，自动把发送改动的已追踪文件纳入 `add`

  ```
  git add -u
  ```

- 自动将所有改动过的被追踪文件一次性提交（若没有新纳入文件，则可以执行这个一步到位）

    ```
    git commit -a -m 'md'
    ```
### (四) 获取远程仓库的更新到本地（见六）

```
git fetch
git rebase
```
或者写作
```
git pull --rebase
```
### (五)  推送

```
git push
```

## 八、异常处理

### (一) 取消 `git` 对某文件的追踪

git 的忽略规则只对未被跟踪的文件有效。如果一个文件已经被纳入版本控制，那么即使在 .gitignore 中添加了忽略规则，git 仍然会继续跟踪这个文件的变化。此时需要进行以下操作：

- 取消跟踪文件，如果是文件夹需要加参数 `-r`
```
git rm --cached aaa.py
git rm --cached -r directory/
```
- 把该文件加入 `.gitignore` 中
### (二) 合并多个`commit` （在没有其他人有新提交打断自己提交记录时）
#### 问题背景
在 `git commit` 的时候只可以提交当前所在路径（可采用 `pwd` 查看）下的子文件，不能提交当前路径未能涵盖住的文件，如 `../path/file.py `。

若某文件夹下全是需要提交的文件，我们一般会选择 `cd` 到该文件上，然后 `git add . + git commit -m 'md'`，对于其他路径下需要提交的文件，可以切换路径后再提交一次。由于有两次未`push`的提交，所以在推送之前需要合并多个`commit`，操作如下：
#### 合并方法
- 最后一个数字控制合并几个commit，写数字 n ，就合并 前 n 个分支
```
git rebase -i HEAD~3
```
- 第一次出现 vim ，将列出前 n 个提交，顺序是最下面的是最新提交，只可以把新提交合并到旧提交中，不可以反过来（这样业符合 git 提交线）
- 把除了第一行以外的 commit 行的 peak 改为 s ，然后  `:wq`。其中， peak 表示保留它作为独立提交的形式， s 表示把当前提交追加到它的提交线上的前一提交中
- 再次出现 vim ，用于提示合并完提交版本之后是否需要修改融合提交版本的描述，一般直接`:q!`
- 通过 `git log` 检查是否合并成功
- 如果本地仓库已经正常合并的话，接下来强制提交到远程仓库，成功后已经 `push` 了的提交也可以合并
```
git push -f
```
> [Note : ]
> 本地仓库合并好了之后，不要再执行 `fetch + rebase` 了，`rebase` 会从远程仓库拉取记录，覆盖掉本地仓库的合并后版本。
### (三)若合并时有冲突

在执行 `git rebase` 时本应自动合并文件内容，但若对于文件的同一行中，远程和本地仓库对它有不同的修改，则 git 会判断两个文件存在冲突，不允许自动合并。与此同时，该文件中会自动标注出冲突地方来：

```
<<<<<<< HEAD
“本地仓库的内容”
=======
“远程仓库的内容”
>>>>>>> master
```

此时只需要删除一种仓库的冲突信息，以及 git 用来辅助标记的符号就好。若本地仓库的是正确的，只保留

```
“本地仓库的内容”
```

反之，只保留

```
“远程仓库的内容”
```

然后再次尝试 `git rebase` 即可

### (四)删除项目的所有 `git` 信息

- 适用情况有：
（1）自己的git已经非常混乱想要重置；
（2）对于 `git clone` 的代码，会自动保存作者的历史提交线。如果希望对该代码进行大幅改动，且改动时完全无需保存改动记录，此时可删除该项目的 `git` 。
- 只需要删除项目根目录下的隐藏文件 `.git` ，项目就会去除所有的版本控制信息，变成一个普通文件夹。
- 对于 LINUX & MAC 需要使用命令删除
```
cd REPO_NAME

ls -la | grep .git
rm -rf .git
```
> **[ Note : ]**
>
> 参数 `-l` 表示长格式显示，可看到文件/文件夹更详细的信息
> 参数 `-a` 则可显示所有文件，包括隐藏文件
> 管道符 `|` 将前一个命令的输出，作为后一个命令的输入。
> `grep` 用于搜索，它不是命令行参数，没有 `--`
- 对于 Windows
文件资源管理器 --> 查看 --> 显示隐藏文件 --> 直接删除 `.git` 文件夹
### （五）`rebase` 或者 `pull` 前本地有未提交的更改
如果本地存在更改，是不允许拉取远程仓库代码的

该情况需要暂存本地更改，然后合并远程仓库代码，再恢复本地更改
```
git stash
git rebase  或者  git pull
git stash pop
```

## 九、通过 `Github CLI` 来避免前往网页版 github ，直接命令行操作一切
- `GitHub CLI` 是一个命令行工具，可将拉取请求、等 GitHub 功能引入终端，使得在一个地方完成所有工作。[官方文档。](https://docs.github.com/zh/github-cli/github-cli)

- macOS 系统上的安装方法

`Homebrew` 是 macOS 上最流行的软件包管理器（也支持 Linux），类似于应用商店- -命令行版

通过`brew --version`观察是否已经安装，若已经安装，使用它安装 `GitHub CLI`。由于  `GitHub CLI` 是由 `GO` 语言开发，因此上述命令会首先下载 GO 包。
```
brew install gh
```
- macOS 上使用 brew 下载失败时的另一种安装方法

通过 Brew 下载很大可能会遇到网络连接断开、版本错误无法构建等等错误

推荐从 `GitHub CLI` 的官方 github 仓库（https://github.com/cli/cli）中下载，[下载地址](https://github.com/cli/cli/releases/tag/v2.80.0)，若是 intel 芯片的 MAC 选择 AMD 的版本，否则选择 arm 版本。下载得到的是一个文件夹，里面包含了`bin`和`share`两个文件夹。

```
# 新建一个专门存放 gh 这种库的文件，将下载内容迁移过去（类似于 win 的移动绿色文件）
sudo mkdir -p /opt/gh/2.80.80
sudo cp downloder/gh_2.80.0_macOS_amd64/* /opt/gh/2.80.80
# 因为系统会自动在 /usr/local/bin/ 中查找命令，把 bin 软链接过去就相当于 win 中的为 bin 文件夹配置 path 变量
# 若不添加软链接过去，则只能在当前文件夹中运行 gh 命令，复制过去就任何路径都可执行 gh
sudo ln -sf /opt/gh/2.80.0/bin/gh /usr/local/bin/gh
```
- Windows 系统上的安装方法

```
winget install GitHub.cli
```
- 检查是否安装成功：`gh --version`

- 首次使用需要登录

```
# 交互式登录方法（最推荐，比通过 token 登录的方法直观）
gh auth login
# 检查是否登录成功
gh auth status
```
- 创建新仓库
```
# 创建公开仓库
gh repo create 仓库名称 --public --description "仓库描述"

# 创建私有仓库
gh repo create 仓库名称 --private --description "仓库描述"
```
## 十、在 github 上存储多个版本的代码
- 首先把版本 1 的代码上传，此时远端仓库（在这里是 github）的分支默认是 `master` ，可以在 github 上面对它重命名，如 `V1.0` ,此后还需要对本地代码进行以下操作：
```
# 重命名本地分支
git branch -m master V1.0
# 从远程仓库获取最新信息
git fetch origin
# 将本地的 V1.0 分支与远程的 origin/V1.0 分支关联起来
git branch -u origin/V1.0 V1.0
git remote set-head origin -a
```
- 重命名版本 2 所在的分支后，再推送
```
# 切换到版本 2 所在的分支（若只有一个分支，这步可省略）
git checkout master2
# 重命名分支名字
git branch -m 新分支名
# 推送该分支
git push -u origin V2.0
```
## 十一、推送大文件
- 若项目中存在某个超过 **100M** 的文件，在推送时会发生报错
```
remote: error: Trace: 25cd697f26a1d86a79634cdb41f48ec39e24f8793ea65e2fe1234c5269c2b0df
remote: error: See https://gh.io/lfs for more information.
remote: error: File _internal.zip is 368.14 MB; this exceeds GitHub's file size limit of 100.00 MB
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
```
- 此时需要用到 lfs ，以下命令检查是否安装它
```
git lfs version
```
- 若未安装，则先安装 LFS
  - Win 系统访问 https://git-lfs.github.com/
  - Mac
  ```
  brew install git-lfs
  ```
  - Ubuntu
  ```
  curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
  sudo apt-get install git-lfs
  ```
  - CentOS
  ```
  curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.rpm.sh | sudo bash
  sudo yum install git-lfs
  ```

- 初始化 LFS

```
git lfs install
```
- 接下来设置 LFS 的跟踪规则
```
git lfs track "*.zip"
# 添加多种后缀
git lfs track "*.exe" "*.pth" "*.bin" "*.dat"
# 查看跟踪规则
git lfs track
```
- 添加了 LFS 规则后会自动生成 `.gitattributes` 文件，需要把它 `add + commit`，然后再推送分支。
## 十二、发布release
- `branch` 中一般存储源代码，若自己在源代码基础上开发了界面，做成了一个他人下载后无需任何代码就可直接使用的程序，则该程序可以打包上传到 github 的 release 部分。
- 通过 git 创建一个 release
```
git tag -a v1.2.3 -m "Release version 1.2.3"
# 或简写，只指定 tag （即版本号）
 git tag v1.2.3
```
- 推送程序至 release，这一步不需要本地分支和 tag 名字相同
```
git push origin v1.2.3
```
- 其实直接在 github 网站中操作还更加简单方便，拖到程序压缩包即可上传
<<<<<<< HEAD

## 十三、一个本地仓库连接多个远程仓库

只要仓库别名不同就可，下面仓库 1 名字为 `origin` ，仓库 2 名字为 `zhiyuan`

```
# 连接仓库 2
git remote add origin https://github.com/zxcvbnmkj/Brain-Dump.git
# 连接仓库 2
git remote add zhiyuan ssh://git@zhiyuanbiji.cn:29492/zhangli2025/essay.git
```

查看本地仓库连接的远程仓库

```
E:\brain_dump\essay>git remote -v
origin  https://github.com/zxcvbnmkj/Brain-Dump.git (fetch)
origin  https://github.com/zxcvbnmkj/Brain-Dump.git (push)
zhiyuan ssh://git@zhiyuanbiji.cn:29492/zhangli2025/essay.git (fetch)
zhiyuan ssh://git@zhiyuanbiji.cn:29492/zhangli2025/essay.git (push)
```

获取指定远程仓库的更新

```
git fetch zhiyuan
```

查看远程文件相对于本地文件的变动。M：修改；D：删除；A：新增。

```
E:\brain_dump\essay>git diff --name-status master zhiyuan/master
A       .DS_Store
D       .gitignore
D       PDM.md
D       assets/image-20251006023557493.png
M       git_use.md
D       java_ruoyi.md
M       mac_skills.md
```

推送到远程指定仓库的指定分支，参数 `-f` 是忽略强制推送，本地分支会完全覆盖远程仓库

```
git push -f zhiyuan master
```

如果只输入 `git push ` 只会推送到默认上游仓库（第一个绑定的）
