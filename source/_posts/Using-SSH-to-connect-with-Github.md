---
title: Using SSH to connect with Github
date: 2022-06-05 22:31:51
tags:
---

# Using SSH to connect with Github


## Creat your own dir for ssh

```bash=
mkdir ~/.ssh

chmod 700 ~/.ssh
```

## Generate your Key

```bash=
cd ~/.ssh

ssh-keygen -t rsa -b 4096 -C "your@gmail.com"

# copy item from id_rsa.pub
cat id_rsa.pub
```

## Set up your Github

![](https://i.imgur.com/QSUS6qb.png)
![](https://i.imgur.com/PpZMrWV.png)
![](https://i.imgur.com/zHo92u9.png)

Paste your content of the id_rsa.pub to here
![](https://i.imgur.com/cFmIzKG.png)



## Test it

```bash=
ssh -T git@github.com
```

```bash=
git@github.com:krnick/your_repo.git
```
