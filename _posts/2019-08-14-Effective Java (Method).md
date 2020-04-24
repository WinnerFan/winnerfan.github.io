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

```
public boolean equals(Object o){
    if(o == this)
        return true;
    if(!(o instance of Person))
        return false;
    Person p = (Person)o;
    return p.name.eqauls(name) && p.id == id;
}
```
### equals于hashCode一起覆盖

- 在同一个应用中，对象equals方法比较的信息未变，则hashCode不变
- 两个对象equals相等，则hashCode值相等
- 两个对象equals不相等，hashCode可相等，但应该不相等

```
基本类型：Type.hashCode(f)
对象引用：递归调用hashCode
数组：数组中没有重要元素，一个非零值；数组为重要元素，Arrays.hashCode
```

### 始终覆盖toString

- 返回对象包含的**所有**重要信息
- 上方注释