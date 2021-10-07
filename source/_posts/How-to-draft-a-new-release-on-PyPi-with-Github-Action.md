---
title: How to draft a new release on PyPi with Github Action
date: 2021-10-07 14:54:14
tags: python, github, release
---

# How to draft a new release on PyPi with Github Action

After updating the version from everywhere in the codebase such as [`__init__.py`](https://github.com/quark-engine/quark-engine/blob/master/quark/__init__.py), [`docs`](https://github.com/quark-engine/quark-engine/blob/245c77c1e1dc9599bae1be0a76a13ebe86c58549/docs/source/conf.py#L25) and [`debian package`](https://github.com/quark-engine/quark-engine/blob/245c77c1e1dc9599bae1be0a76a13ebe86c58549/debian/control#L20), you can click "Releases" for drafting a new version release.

![](https://i.imgur.com/c7JO72j.png)

After that, you can click the "Draft a new release".

![](https://i.imgur.com/bnApXxl.png)

The next step is to fill out the version number you want to release, and the changelog from the last release to now.

![](https://i.imgur.com/CzMaMTw.png)

After completing the information for this version, you can click "Publish release" to publish this version.

![](https://i.imgur.com/B3DXFpW.png)

Once this version is released, [Github Action](https://github.com/quark-engine/quark-engine/actions/workflows/pythonpublish.yml) will initiate a process to publish it on PyPi automatically.

![](https://i.imgur.com/mbMR7Uo.png)

:::info
Please note that there is a limitation that you cannot upload twice with the same release version number.


