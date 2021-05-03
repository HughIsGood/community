# 牛客网社区系统项目
## 项目介绍
本项目实现了牛客网社区系统的部分功能：讨论区。这个项目的整体结构来源于牛客网，
主要使用了Springboot、Mybatis、MySQL、Redis、Kafka等工具。主要实现了用户的注册、登录、发帖、点赞、系统通知、按热度排序、搜索等功能。
另外引入了redis数据库来提升网站的整体性能，实现了用户凭证的存取、点赞关注的功能。
基于 Kafka 实现了系统通知：当用户获得点赞、评论后得到通知。
利用定时任务定期计算帖子的分数，并在页面上展现热帖排行榜。
## 该项目的用途
本项目用于夯实基础知识，旨在通过本项目加深对SSM框架的理解。同时，通过开发小小的社区网站，熟悉项目开发流程。最终目的--》秋招找一个好工作！！！
## 目录


1. [注册登录](#1)
2. [拦截器](#2)
3. [发帖](#3)
4. [Spring事务的管理方式](#4)
5. [添加评论](#5)
6. [处理异常](#6)
7. [点赞](#7)
8. [利用kafaka实现发送系统通知功能](#8)
9. [Spring ThreadPoolTaskSchedular实现定时任务](#9)
10. [热帖排行](#10)

<span id="1"></span>
## 注册登录
