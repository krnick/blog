---
title: Pipenv usage
date: 2020-09-24 14:15:06
tags:
- python
- git
---

# Install from github
```bash=
pipenv install -e git+https://github.com/quark-engine/quark-engine.git#egg=quark-engine
```
* First "quark-enigne": user
* Second "quark-engine": repo
* @develop: The branch what you want to install
* #egg=quark-engine: The package name that will be recorded in pipenv
