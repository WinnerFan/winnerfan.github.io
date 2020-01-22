---
layout: post
title: Effective Java (Method)
tags: Java
---
## 对于对象通用的方法

### equals时注意
- 自反性 x != null时，x.equals(x) == true
- 对称性 x.equals(y) == true时，y.equals(x) == true
- 传递性 x.equals(y) == true, y.equals(z) == true时，x.equals(z) == true
- 一致性 多次调用，结果相同
- x ！= null时，x.equals(null) == false
