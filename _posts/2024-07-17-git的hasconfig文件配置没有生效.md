---
sharetitle: 2024-07-17-git的hasconfig文件配置没有生效
share: "true"
title: git的includeIf hasconfig文件配置没有生效
date: 2024-07-17
date modified: 2024-07-17
tags:
  - issue
  - git
---

以前在windows的git中配置了差异化配置文件，用于在工作项目中区分提交邮箱，但最近用wsl(ubuntu)提交发现没有生效

`.gitconfig`如下

```markdown
[user]
  name = My Name
  email = email@personal.com

# It is important the includeIf comes after the user configuration section
# so that the email in the work config is picked up as the email to use
[includeIf "hasconfig:remote.*.url:https://github.com/work_org/**"]
  path = .gitconfig_work

# SSH remote to cover all bases
[includeIf "hasconfig:remote.*.url:git@github.com:work_org/**"]
  path = .gitconfig_work
```

 在.gitconfig_work中

```markdown
[user]
  email = email@work.com
```

在我的仓库路径正确的前提下，即

```bash
➜  github git:(master) git config --get remote.origin.url
https://github.com/work_org/some_repo
```

查找配置并没有找到我在gitconfig_work中配置的信息

```bash
➜  github git:(master) git config --get-all user.email 
email@personal.com
(END)
```

但是预期结果应该是

```bash
➜  github git:(master) git config --get-all user.email 
email@personal.com
email@work.com
(END)
```

经搜索发现，是由于git版本过低，windows中为2.41.0，wsl中版本为2.34.1，但是`includeIf hasconfig:` 是在2.36.0以后才有的feature

因此在ubuntu中对git 进行更新即可

```bash
git clone https://github.com/git/git .core-git &&
cd .core-git &&
git switch -d v2.37.3 &&
sudo apt-get update &&
sudo apt-get install git gcc make libssl-dev libcurl4-openssl-dev \
  libexpat-dev tcl tk gettext git-email zlib1g-dev \
  checkinstall &&
printf '%s\n' prefix=/usr gitexecdir=/usr/lib/git-core NO_TCLTK=1 >>config.mak &&
make -j8 &&
eval "$(tr -d ' ' <GIT-VERSION-FILE)" &&
sudo checkinstall -y --pkgname=git --pkgversion=2:$GIT_VERSION \
  --provides=git,\ git-man,\ git-email \
  --replaces=git,\ git-man,\ git-email \
  make -j8 install
```
