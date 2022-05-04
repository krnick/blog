---
title: Cheat sheet for poetry
date: 2022-05-04 14:05:07
tags:
---

# Cheat sheet for poetry



## Init project

```bash=
poetry init
```


## Open existing project

```bash=
poetry install
```


## Build shell

```bash=
poetry env use python3
```

## Active

```bash=
poetry shell
```

## New Package

```bash=
poetry add black

poetry add pendulum@^2.0.5
poetry add "pendulum>=2.0.5"
```


## Remove Package


```bash=
poetry remove
```



## New Package -dev

```bash=
poetry add black -D
```

## Lock Package

```bash=
poetry lock
```



## Export

```bash=
poetry export -f requirements.txt -o requirements.txt --without-hashes --dev
```
