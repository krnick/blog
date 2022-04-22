---
title: Github Bot
date: 2022-04-22 14:56:57
tags:
---

# Github Bot



Register [Heroku](https://signup.heroku.com)


## Local setting

```
pip install gidgethub
```

get your Github Token


In Unix / Mac OS:
```
export GH_AUTH=your token
```

## Local test

```python=
import asyncio
import os
import aiohttp
from gidgethub.aiohttp import GitHubAPI

async def main():
    
   async with aiohttp.ClientSession() as session:
       gh = GitHubAPI(session, "krnick", oauth_token=os.getenv("GH_AUTH"))
       
       # 開啟新的 issue
       await gh.post('/repos/krnick/Github-Bot/issues',
             data={
                 'title': 'Hello Nick',
                 'body': 'This is github bot from JunWei',
             })
             
             
        # 關閉 issue
       await gh.patch('/repos/krnick/Github-Bot/issues/1',
             data={'state': 'closed'},
             )

loop = asyncio.get_event_loop()
loop.run_until_complete(main())

```



## deploy

create a new APP
![](https://i.imgur.com/gzBhd98.png)
auth
![](https://i.imgur.com/JBnDaeP.png)
connect Github
![](https://i.imgur.com/yGLo7zQ.png)

![](https://i.imgur.com/Ni8AacE.png)


![](https://i.imgur.com/P2iA4fc.jpg)
![](https://i.imgur.com/KNm6cJ5.png)

---
Creat a new webhook at the repo you want to build

![](https://i.imgur.com/MpcbHjw.png)
![](https://i.imgur.com/jhyt9Zj.png)
![](https://i.imgur.com/FyljvND.png)

Set your key at Heroku


GH_AUTH

GH_SECRET



## typo check repos example

![](https://i.imgur.com/1LdIE7z.png)
![](https://i.imgur.com/zUQiEZn.png)
![](https://i.imgur.com/7G8Zg58.png)
![](https://i.imgur.com/wMZShV5.png)
![](https://i.imgur.com/PtDjJze.png)
![](https://i.imgur.com/DPPEXv2.png)
![](https://i.imgur.com/AgmG33A.png)


## Ref

https://github-bot-tutorial.readthedocs.io/en/latest/index.html
