---
title: 登录OAF页面和FORM的职责不对称
date: 2019-06-27 10:31:12
tags:
- Oracle
- E-Business Suite
- R12.x
categories:
- Oracle
- E-Business Suite
---
<i class="fa fa-question-circle"></i> **问题：** 
登录OAF页面和FORM的职责不对称，即OAF登录页看见的职责条目不等于登录FORM后的职责条目。
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/20190627103358.png)

<i class="fa fa-arrow-circle-right"></i> **方法：** 
打开系统管理员职责，刷新工作流程请求 ***工作流职责层次结构传递***。
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/20190627104224.png)