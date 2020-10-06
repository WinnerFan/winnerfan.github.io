---
layout: post
title: Standard Java
tags: Java
---
## 命名

1. int数组表示一组数据，用类封装；`int[STATUS_VALUE] == FLAGGED`替代为`cell.isFlagged()`，掩盖魔术数
2. accountList除非真是List，否则可以用accounts，accoutGroup
3. 避免数字和废话，`copy(char a1[], char a2[])`改为`copy(char source[], char destination[])`；Product类和ProductInfo、ProductData类
4. 类名对象名为名词
5. 方法名为动词，使用描述参数的静态工厂方法名`Complex.FromRealNumber(23)`优于`new Complex(23)`

## 函数
1. 短小
2. 只做一件事，例如checkPassword函数中调用了Session.initialize()，产生了副作用初始化会话
3. 参数不要大于2，否则换为对象
4. 分割指令与查询，例如`if(attributeExists("username")){ setAttribute("username","a");...}`
5. 异常代替错误码
6. `try/catch`抽离，后面跟一个函数
7. 不传null值

## 注释

1. 用代码来阐述`if(employee.isEligibleForFullBenefits())`
2. 好：法律信息、信息注释、意图解释、警示、TODO应该做但现在还没做的、Javadoc
3. 坏：喃喃自语、多余注释、每个函数Javadoc、日志式
