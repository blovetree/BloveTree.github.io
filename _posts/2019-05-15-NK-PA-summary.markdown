---
layout:     post
title:      "NK-PA"
subtitle:   "Summary"
date:       2019-05-15
author:     "Dylan"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - PA
    - Summary
---



# 前言

> PA是学习system的引子

NK-PA应该是从4月初到5月初。做PA的主要困难大概在于对计算机系统的认识，以及写代码

学习计算机可以自底向上，也可自顶向下，甚至是只学一部分。无论怎样，学习系统更重要的可能就是反复体会

PA的代码很复杂，我用的是understand，我记得是可以显示函数调用的蝶形图



# 结构 

```mermaid
graph BT;
    L1("硬件")-->L2("硬件抽象");
    L2-->L3("系统软件");
    L3-->L4("应用");
```

```mermaid
graph LR
NDB("NDB")--d1("调试器")
NEMU("NEMU")--d2("模拟器")
```


# 链接 

[NK-PA-NEMU](https://blovetree.github.io/2019/05/15/NK-PA-NEMU/)

[NK-PA-NEXUS-AM](https://blovetree.github.io/2019/05/15/NK-PA-NEXUS-AM/)

[NK-PA-NANOS-LITE](https://blovetree.github.io/2019/05/15/NK-PA-NANOS-LITE/)

[NK-PA-NAVY-APPS](https://blovetree.github.io/2019/05/15/NK-PA-NAVY-APPS/)