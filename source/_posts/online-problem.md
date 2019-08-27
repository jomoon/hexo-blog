---
title: online-problem
date: 2019-08-26 16:29:05
categories:
- 线上
- 问题
---

# 线上问题的记录

## 某次更新后，线上经常cpu打满

```bash
    apt-get install procps

    top

    top -H -p  ${pid}

    jstack ${pid} | grep -A 10 0x5bd8

```