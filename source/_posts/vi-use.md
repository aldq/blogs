---
title: vi-use
date: 2023-07-04 15:13:18
tags:
---

## 批量替换

```bash
将文件tihuan（假设此文本中字符a）中的所有字符a换成字符w，其命令为：
1。vi tihuan
2。按esc键
3。按shift+：
4。在：后输入    %s/a/w/g

其中s为：substitute，%表示所有行，g表示global

如果要替换34到78行之间的，则如下：
前几步同上，最后一步为：
:34,78s/a/w/
```

<!--more-->

## aa

## bb
111
