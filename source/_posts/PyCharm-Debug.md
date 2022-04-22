---
title: PyCharm Debug
date: 2022-04-22 15:11:48
tags:
---

# PyCharm Debug

* step over 單步執行

在函數內遇到子函數不會執行


* steop into 單步執行

在函數內遇到函數會跳進去


* step into my code

遇到子函數就進入並且繼續單步執行，不會進入到原始碼中

* step out 跳出一個函數


如果進到一個函數內，不想看了，即可按這個


以上四個功能，就是最常用的功能，一般操作步驟就是，設置好斷點，debug運行，然後F8 單步調試，遇到想進入的函數F7 進去，想出來在shift + F8，跳過不想看的地方，直接設置下一個斷點，然後F9 過去。
