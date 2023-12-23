# git 服务器设置


1. 登录到服务器

选择一个目录 作为后续git 使用的路由

```bash

groupadd git
useradd git

cd  /srv/git

git init  --bare crmservice.git


```


2. 在客户电脑端使用

```bash
git init

git remote add origin git@server:/srv/git/crmservice.git




```

<p>


## 问题


这里最初的想法是 想基于git hook 实现代码的自动编译和运行

但是当前卡在了远程仓库没有代码无法自动打包执行

验证了git hooks 目录下执行post-receive脚本可以在 当前git 目录下执行

```bash

.
├── branches
├── config
├── description
├── HEAD
├── hooks
    ├── applypatch-msg.sample
    ├── commit-msg.sample
    ├── fsmonitor-watchman.sample
    ├── post-receive
    ├── post-update.sample
    ├── pre-applypatch.sample
    ├── pre-commit.sample
    ├── pre-merge-commit.sample
    ├── prepare-commit-msg.sample
    ├── pre-push.sample
    ├── pre-rebase.sample
    ├── pre-receive.sample
    └── update.sample
├── info
├── objects
├── post-receive.txt
└── refs

```


post-receive文件
```sh
#!/bin/sh

echo "testtest" > post-receive.txt

branch_name=$(git rev-parse --symbolic --abbrev-ref $refname)

echo $branch_name  >> post-receive.txt

```

提交后执行效果为 在 crmservice.git 文件夹下存在文件

post-receive.txt

内容为

```bash

[root@iZ2ze6psmhg970hbukeh0tZ crmservice.git]# cat post-receive.txt
testtest

```


### 问题突破

[找到文章](https://www.cnblogs.com/wowchky/p/9177036.html)



```bash
# 寻找一个工作目录 clone 仓库

git clone /srv/git/crmservice.git

# 然后在当前目录就会出现一个 带有代码的仓库

```

post-receive 文件如下

```bash


#!/bin/sh




#判断是不是远端仓库

IS_BARE=$(git rev-parse --is-bare-repository)
if [ -z "$IS_BARE" ]; then
echo >&2 "fatal: post-receive: IS_NOT_BARE"
exit 1
fi


unset GIT_DIR
DeployPath="/root/crm/backend/crmservice"
GitPath=$DeployPath/.git
cd DeployPath
git pull /srv/git/crmservice.git

chown git:git -R $DeployPath

# 拿到运行的run 程序的pid
RunPid=$(ps -ef |grep run |head -n 1 | awk '{print $2}')

kill -STOP $RunPid
rm run -y

go build -o run main.go


./run

```

失败了 发现没有生成 run 可执行文件 不知道哪儿去看报错日志

去hooks目录下 手动执行失败 发现服务器环境没有 go环境



**后面忽然发现  在git push 的地方会有脚本报错日志**

```bash

$ git push
git@8.146.209.146's password:
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 12 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 447 bytes | 447.00 KiB/s, done.
Total 4 (delta 2), reused 0 (delta 0), pack-reused 0
remote: hooks/post-receive: line 20: cd: DeployPath: No such file or directory
remote: fatal: this operation must be run in a work tree
remote: chown: cannot access '/root/crm/backend/crmservice': Permission denied
remote: hooks/post-receive: line 31: kill: (1154) - Operation not permitted
remote: rm: invalid option -- 'y'
remote: Try 'rm --help' for more information.
remote: hooks/post-receive: line 35: go: command not found
remote: hooks/post-receive: line 38: ./run: No such file or directory
To 8.146.209.146:/srv/git/crmservice.git
   7f9f7a0..044ea88  dev -> dev


```

