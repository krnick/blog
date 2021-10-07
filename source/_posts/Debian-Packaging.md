---
title: Debian Packaging
date: 2021-05-03 16:10:42
tags:
- Debian
- Linux
- Kali Linux
---

[Kali Linux issue 0007121](https://bugs.kali.org/view.php?id=7121)

# Kali Linux Release

## 1. Prepare your Kali Linux virtual machine to build the Debian package.

- [Virtualbox](https://www.virtualbox.org/wiki/Downloads)
- [Kail Linux image for virtualbox](https://www.kali.org/get-kali/#kali-virtual-machines)

### Update your kali Linux package source

```bash
echo "deb http://http.kali.org/kali kali-rolling main non-free contrib" | sudo tee /etc/apt/sources.list

sudo apt-get update
```

After setting up your virtual machine, you have to install the required packages based on the official documentation on this [website](https://www.kali.org/docs/development/setting-up-packaging-system/).


### Install the required dependencies

```bash
sudo apt install -y packaging-dev apt-file gitk mr

sudo apt-get install -y devscripts debhelper dh-make git-buildpackage sbuild dh-python python3-all
```

### Set up sbuild and log in again after completion

```
sudo mkdir -p /srv/chroots/

cd /srv/chroots/

sudo sbuild-createchroot --keyring=/usr/share/keyrings/kali-archive-keyring.gpg --arch=amd64 --components=main,contrib,non-free --include=kali-archive-keyring kali-dev kali-dev-amd64-sbuild http://http.kali.org/kali

echo "source-root-groups=root,sbuild" | sudo tee -a /etc/schroot/chroot.d/kali-dev-amd64-sbuild*

sudo sbuild-adduser $USER
```

## 2. Creating the Debian files

The Debian package needs that the following four files are required in the debian directory.

* control
* copyright
* changelog
* rules

[Here](https://www.debian.org/doc/manuals/maint-guide/dreq.en.html) is the official documentation on how to write all four files.

## 3. Importing the Quark project

Please replace the version number you would like to update.

```bash
mkdir -p ~/kali/packages/quark-engine ~/kali/upstream/

wget https://github.com/quark-engine/quark-engine/archive/refs/tags/v21.4.3.tar.gz  -O ~/kali/upstream/quark-engine_21.4.3.orig.tar.gz

cd /home/kali/kali/packages/quark-engine

git init

gbp import-orig ~/kali/upstream/quark-engine_21.4.3.orig.tar.gz
```


## 4. Build Package

```bash
gbp buildpackage --git-builder=sbuild
```

Then you will have the Quark `.deb` file if everything goes well.

## 5. Request package upgrade

1. Register [Kali Linux Bug Tracker](https://bugs.kali.org/my_view_page.php)

2. Report an issue

Report an issue by clicking the "Report Issue"

![](https://i.imgur.com/R7fGbSY.png)

3. Fill out the form

Please choose the "Tool upgrade request" option in `category` and "kali-dev" option in `product version` and keep anything else default value.

![](https://i.imgur.com/Wclavqq.png)

The next step is filling out the summary and the description.

4. Attach the .deb file

Upload the `.deb` file you just created earlier.

![](https://i.imgur.com/5Mbu3hf.png)

5. Submit the issue

When everything is all set, you can click "Submit issue".


https://www.kali.org/docs/development/public-packaging/
https://www.kali.org/docs/development/intro-to-packaging-example/
https://www.kali.org/docs/development/setting-up-packaging-system/
https://www.kali.org/docs/development/advanced-packaging-example/