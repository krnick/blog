---
title: 'Unicode, ASCII, and why UTF-8'
date: 2023-05-09 21:01:54
tags:
---
# Unicode, ASCII, and why UTF-8

## 歷史

很久以前，只有 ASCII ，當時所使用的符號並不多，所以可以透過 ASCII 來做一對一的符號對照表， ASCII 用了 7 個 bits 來表示 128 個 `character`，包括大寫英文字母、小寫英文字母、以及其他符號。

然而，隨著其他語言的興起，ASCII 已經不能再完全表達所有語言的符號，例如中文、日文、韓文等等，因此 Unicode 出現了作為解決方案。Unicode 這套對照表可以對照世界上所有的符號，例如 "你" 的 Unicode 編碼就是 "\u4f60"，而這個 \u4f60 稱之為 Unicode code point。

Unicode 解析編碼有幾種，包括 UTF-32、UTF-16、UTF-8。其中，UTF-32 使用固定 32 bits 或 4 bytes 的空間來儲存，依此類推，但最大的問題是會有空間浪費的問題。因此，最常被使用的解析編碼方式是 UTF-8。UTF-8 則是一種長度可變的編碼方式，可以同時解析介於 8 bits 到 32 bits 長度的字元（1 byte 到 4 bytes），對於 ASCII 則是 1 byte 也就代表可以兼容 ASCII。最高位元的 bit 代表接下來的 UTF-8 使用了幾個 byte。

當我們使用 \x 表示一個字元時，後面的十六進制數字表示這個字元在 UTF-8 編碼中所佔用的字節的值。例如，\x41 表示 ASCII 範圍內的大寫字母 A，因為它的 UTF-8 編碼只需要一個字節，而這個字節的值恰好是 41（十六進制）。

## 格式

0xxxxxxx                              -> 0-127 ASCII
110xxxxx 10xxxxxx                     -> 128-2047
1110xxxx 10xxxxxx 10xxxxxx            -> 2048-65535
11110xxx 10xxxxxx 10xxxxxx 10xxxxxx   -> 65536-0x10ffff

\u 16 bit unicode (\uhhhh)
\U 32 bit unicode (\Uhhhhhhhh)

## 例如

* 世界
* "\u4e16\u754c" 為 `世界` 的 \u unicode code point 表示法
* "\xe4\xb8\x96\xe7\x95\x8c" 為 `世界` 的 \x unicode code point 表示法


其中，若將"\u4e16"轉換為單一 byte 的表示方式，轉換流程如下：

1. 先將"\u4e16"轉換成2進位表示 -> 0100111000010110
1. 依照範圍，0100111000010110 需要使用3個bytes，也就是 1110xxxx 10xxxxxx 10xxxxxx
1. 將對應的2進位 -> 0100111000010110，依序填入 xxx 中
1. 得到結果：11100100 10111000 10010110
1. 再將此2進位表示轉換為16進位 -> E4B896
1. 因此，"\u4e16"在UTF-8編碼下所對應的bytes為3，並且分別為"\xE4"、"\xB8"和"\x96"。

Ref:
https://www.youtube.com/watch?v=ut74oHojxqo&ab_channel=StudyingWithAlex
