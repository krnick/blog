---
title: Debug Golang with VSCode
date: 2023-05-01 16:19:35
tags:
---

# Debug Golang with VSCode


## Example

### Create a simple go lang file with the below code:


* main.go

```go
package main

import (
	"fmt"
	"os"
	"strings"
)

func main() {
	fmt.Println("hello world")

	name := os.Args[1:]

	fmt.Println("hello " + strings.Join(name, ""))
}
```

![](https://i.imgur.com/bGNM5Od.png)

### Make sure the go.mod is initialized

* go.mod

```go
module test-debug

go 1.19
```


it will print the args from the command-line args


### Switch to the `Run and Debug`

And choose the `create a launch.json` file and fill in the config for debugging

![](https://i.imgur.com/RgJ5fZw.png)

![](https://i.imgur.com/b17ir3J.png)

![](https://i.imgur.com/LgB5Pfn.png)

The config example is below:

```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch Package",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "program": "${fileDirname}",
            "args": ["nick","ben"]
        }
    ]
}
```

the **args** are the arguments you want to put after the go program, for example:

`go run main.go nick ben`

then the "**args**" is:

`"args": ["nick","ben"]`


### Now you can start your debugging

![](https://i.imgur.com/KSj1CPp.png)

![](https://i.imgur.com/ru9VdPO.png)

