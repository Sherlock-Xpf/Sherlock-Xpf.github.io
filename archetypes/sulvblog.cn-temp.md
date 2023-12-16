---
title: "{{ replace .Name "-" " " | title }}" #标题
date: {{ .Date }} #创建时间
lastmod: {{ .Date }} #更新时间
author: ["Sulv"] #作者
categories: 
- 分类1
- 分类2
tags: 
- 标签1
- 标签2
description: "" #描述
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true #是否展示评论
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: "" #图片路径：posts/tech/文章1/picture.png
    caption: "" #图片底部描述
    alt: ""
    relative: false
---

————————————————
版权声明：本文为「Sulv's Blog」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://www.sulvblog.cn/posts/blog/build_hugo/#4%e9%85%8d%e7%bd%ae%e6%96%87%e4%bb%b6