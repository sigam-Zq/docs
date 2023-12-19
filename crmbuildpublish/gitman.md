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

其他正常使用操作就可以了