---
title: javascript-00 简述
date: 2016-04-25 19:54:27
tags: [javascript]
---


# 简述
现在开始学习javascript！根据大神的推荐，阅读《JavaScript DOM编程艺术(第2版)一书入门》

# JavaScript简史
JavaScript是网景公司开发的一门和Java没什么关系的语言，使得网站的内容不再局限于枯燥的文本，交互性得到显著改善，在与微软公司竞争中，网景公司联合ECMA对JavaScript语言进行了标准化，于是出现了ECMAScript，同一个语言的另一个名字。
和Java不同，JavaScript是一种脚本语言，**只能通过Web浏览器解释执行**(还有node.js?)，相对的特点是简单。

# DOM
DOM(Document Object Model，文档对象模型)是一套对文档的内容进行抽象和概念化的方法。
例如：
```
document.images[2] #文档中的第三个图像
document.forms['details'] #文档中名为details的表单
```

# 制定标准
应为微软和网景浏览器之间的冲突，程序员在编写DOM脚本时必须知道要运行在哪种浏览器，令人痛苦难堪。
W3C结合大家的优点推出了一个标准化DOM，可以让**任何一种程序设计语言**对**任何一种标记语言**编写出来的**任何一份文档**进行操控。

DOM是一种API，一个与系统平台和编程语言无关的接口，不同的浏览器通过不同的方式完成相同的任务。

苹果公司2003年发布Safari基于WebKit，WebKit是一个开源Web浏览器引擎。

DOM把HTML/SHTML、CSS和JavaScript凝聚在一起
