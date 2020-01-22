---
layout: post
title: Python Basic
tags: Python
---
## Python

### 字符串和数字

方法|作用|备注
:-:|:-:|:-:
.title()|首字母大写|
.upper()|全部大写|
.lower()|全部小写|
.rstrip()|去除末尾换行、回车、制表符、空格|
.lstrip()|去除开头换行、回车、制表符、空格|
.strip()|去除首尾换行、回车、制表符、空格|
.split()|换行、回车、制表符、空格分割|
+|字符串拼接|
==|判断相等|区分大小写
!=|判断不等|区分大小写

- 单引号和双引号均可，但不可以和值中冲突
	- Right: msg = "One of Python's"
	- Wrong: msg = 'One of Python's'
- 加、减、乘、除、乘方、余对应+、-、*、/、**、%
- 数字转字符串str(a)
- 字符串转数字int(a)、float(a)
- 字符串输入a = input("msg")，a非msg而是输入的值
- 命名：**类名驼峰，每个单词首字母大写；实例名和模块名小写格式，下划线连接**

### 列表，a=[]

方法|作用|备注
:-:|:-:|:-:
a[0]|访问第一个元素|
a[-1]|访问最后一个元素|
.append(b)|末尾添加b|
.insert(0,b)|在0位置添加b|
del a[0]|删除a中0位置元素|
.pop(0)|弹出索引为0的元素|省略0，默认弹出末尾值
.remove(b)|去除元素值为b的元素|b单双引号均可，只是删除第一个指定值
.sort()|永久排序|
sorted(a)|临时排序|
.reverse()|逆序|
len(a)|列表长度|
range(1,5)|创建数组[1,5)|
min(a)|a中最小值|可用于字符串
max(a)|a中最大值|可用于字符串
sum(a)|a的和值|
'aaa' in a|判断存在|返回True或False
'aaa' not in a|判断不存在|返回True或False
and|和|多个条件
or|或|多个条件

- 循环判断，注意**冒号**
```
# 判断非空
if a:
	for aloop in a:
		if aloop == 'aa':
			print(aloop)
		elif aloop == 'aaa':
			print(aloop)
		else:
			print('this is not aa or aaa')
# 另一种循环
print([val*2 for val in a])
```
- 部分列表
```
# 索引0-2
a[0:3]
# 索引0-4
a[:5]
# 索引-2,-1
a[-2:]
```
- 复制列表
```
# 深复制
b = a[:]
# 浅复制
b = a
```
- 元组，不能**单独**改变一个值
```
a = (1,2)
# right
a = (3,5)
# wrong
# a[0] = 3
```

### 字典，a={}

- 增删改查
```
a['key1'] = 'val1'
del a['key1']
a['key1'] = 'value1'
a['key1']
```
- 遍历

	```
	for key, val in a.items():
		print(key+": "+val)
	
	# sorted(a.keys())排序后
	for key in a.keys():
		print(key)
	
	# set(a.values())去重后
	for val in a.values():
		print(val)
	```

### 函数

- 固定参数，具有初始值时调用参数个数可调，副本传递
	
	```
	# 默认值为b，不能有空格
	def fun(grade, name='b'):
		print("Hello "+grade+name)
		person = {'name':name, 'grade':grade}
		return person
	# 先列出没有默认值，后列出可选参数即有默认值的
	person1 = fun('1', 'a')
	person2 = fun('2')
	
	# 禁止修改列表时，可以传递列表副本
	fun(a[:])
	```

- 随机参数
```
def user_profile(first, last, **user_info)
	profile={}
	profile['first_name'] = first
	profile['last_name'] = last
	for key, val in user_info.items():
		profile[key] = val
	return profile

user_profile('a','b',location='c',field='d')
```

- 导入模块、函数或类
	- import filename as f<br>
	  f.function()
	- from filename import function as fun<br>
	  fun()
    - from filename import *<br>
      function()

### 类
- 普通类，方法修改类属性还可以添加逻辑

	```
	class Car():
		def __init__(self, make, model)
			self.make = make
			self.model = model
			self.odometer_reading = 0
		# update中可以添加逻辑
		def update_odometer(self, mileage)
			if mileage >= self.odometer_reading:
				self.odometer_reading = mileage
		def read_odometer(self)
			print(str(self.odometer_reading))
	
	my_car = Car('audi','R8')
	my_car.update_odometer(10)
	```

- 继承，可以重写方法

	```
	class Son(Father):
	
	#子类重写方法
	def action(self)
		super().action
	#子类重写私有方法
	def __action(self)
		print("father")
	def action(self)
		super()._Father__action()
	#子类自己定义__init__方法后，访问父类属性需要
	def __init__(self)
		super().__init__()
	```

- 实例作为属性，同时引入了方法
```
class Battery()
def __init__(self, battery_size=70):
	self.battery_size = battery_size
def descirbe_battery(self)
	print(str(self.battery_size))

class ElecticCar(Car):
	def __init__(self, make, model)
		super().__init__(make, model)
		self.battery_size = Battery()

my_tesla = ElecticCar('tesla', 'model s')
my_tesla.battery.describe_battery
```
### 文件

- read()到达文件末尾返回一个空字符串，显示为一个空行
- 每行末尾都有一个看不到的换行符，print又会增加一个换行符
```
with open('pi_digits.txt') as file_object:
	contents = file_object.read()
	print(content.rstrip())
	
	for line in file_object:
		print(line.rstrip())
```
- open(filename, 'w') r只读(default) w写入 a附加 r+读写
```
with open('', 'w') as file_object:
	file_object.write("write")
```
- try-except 抛异常，excpt中pass则跳过  
```
try:
	answer = int(first)/int(second)
except ZeroDiversionError:
	print("error")
else:
	print(answer)
```

-json
```
import json
json.dump(target,file_object)
json.load(file_object)
```

### 测试

```
import unittest
class ClassNameTest(unittest.TestCase)
	# by setUp
	def setUp(self)
		self.my_test = TestClass
		self.test_variable = 'a'
	def by_setUp_method(self)
		slef,my_test.for_test_method(test_variable)
	# normal
	def normal_method(self)
		a = for_test_method('a')
		self.assertEqual(a,'a')

unittest.main()
```
- setUp(self), set the variables or the objects
- assert
	- assertEqual(a,b)
	- assertNotEqual(a,b)
	- assertTrue(x)
	- assertFalse(x)
	- assertIn(a,list)
	- assertNotIn(a,list)