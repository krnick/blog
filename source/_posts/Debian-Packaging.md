---
title: Debian Packaging
date: 2021-05-03 16:10:42
tags:
- Debian
- Linux
- Kali Linux
---

前陣子要將 Quark 上傳至 [Kali Linux](https://www.kali.org/)，需要包成一個 Debian 的安裝檔案費了不少力氣，以下紀錄一下該如何打包一個 Python 專案至 Debian 的 .deb 安裝檔案。

[Kali Linux issue 0007121](https://bugs.kali.org/view.php?id=7121)

## Creating the Debian files

Debian套件強制規定 debian 目錄下需要有以下四個檔案

* control
* copyright
* changelog
* rules

細節可參考 Quark 完成後的 [Debian-目錄](https://github.com/quark-engine/quark-engine/tree/master/debian)

---

### 產生 debian 目錄

使用 `dh_make` 可以自動幫你產生所需的這四個檔案，`quark-engine_21.02.2.orig.tar.gz` 為你的專案壓縮檔案。

```bash=
dh_make -p quark-engine_21.02.2 -f quark-engine_21.02.2.orig.tar.gz 

rm *.ex *.EX README.* *.docs
```

## System Requirements


更新環境

```bash=
echo "deb http://http.kali.org/kali kali-rolling main non-free contrib" | sudo tee /etc/apt/sources.list

sudo apt-get update
```

安裝需要的檔案
```bash=
sudo apt install -y packaging-dev apt-file gitk mr

sudo apt-get install -y devscripts debhelper dh-make git-buildpackage sbuild dh-python python3-all
```

設定 sbuild，完成後重新登入

```bash=
sudo mkdir -p /srv/chroots/
cd /srv/chroots/

sudo sbuild-createchroot --keyring=/usr/share/keyrings/kali-archive-keyring.gpg --arch=amd64 --components=main,contrib,non-free --include=kali-archive-keyring kali-dev kali-dev-amd64-sbuild http://http.kali.org/kali

echo "source-root-groups=root,sbuild" | sudo tee -a /etc/schroot/chroot.d/kali-dev-amd64-sbuild*

sudo sbuild-adduser $USER
```


## Importing

匯入你的專案

```bash=

mkdir -p ~/kali/packages/quark-engine ~/kali/upstream/

wget https://github.com/quark-engine/quark-engine/archive/refs/tags/v21.4.3.tar.gz  -O ~/kali/upstream/quark-engine_21.4.3.orig.tar.gz

cd /home/kali/kali/packages/quark-engine

git init

gbp import-orig ~/kali/upstream/quark-engine_21.4.3.orig.tar.gz
```

## Build Package

產生 `.deb` 檔案

```bash=
gbp buildpackage --git-builder=sbuild
```

## Reference

https://www.kali.org/docs/development/public-packaging/
https://www.kali.org/docs/development/intro-to-packaging-example/
https://www.kali.org/docs/development/setting-up-packaging-system/
https://www.kali.org/docs/development/advanced-packaging-example/