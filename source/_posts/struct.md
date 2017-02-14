---
title: struct
date: 2017-02-13 13:07:26
tags: [python]
categories: python 学习
---
## Python struct 用法
### 主要函数
* pack(fmt, v1, v2, ...)
根据所给的fmt描述的格式将值v1，v2，...转换为一个字符串
* unpack(fmt, bytes)
根据所给的fmt描述的格式将bytes反向解析出来，返回一个元组
*  calcsize(fmt)
根据所给的fmt描述的格式返回该结构的大小

### 主要定义格式
* i -- int
* f -- float
* d double
*  s string
*  其他可看官方文档
<!-- more -->

### 实例
* pack
``` python
feature = [1.0,2.0,3.0]
buf = struct.pack('%if' % len(feature), *feature)
```
* 加上文件
```python
length = 3
with open('test.txt','wb') as fw:
     string = ''
     string += struct.pack("i", length)
     fw.write(string)
```
* unpack
```python
feature_lenght = 10
 with open('test.txt', 'rb') as f:
     feature_tuple = struct.unpack('%if' %feature_length, f.read())
```
* calcsize
```python
struct.calcsize('ii') #返回8
```

### struct 处理结构体
* 需要处理的结构体
```cpp
struct header
{
unsigned short  usType;
char[4]         acTag;
unsigned int    uiVersion;
unsigned int    uiLength;
};
```
* python 解析此struct
``` python
str = struct.pack('B4sII', 0x04, 'aaaa', 0x01, 0x0e)
```
* unpack 此结构体
``` python
type, tag, version, length = struct.unpack('B4sll', str)
```

### 常用序列化工具
#### cPickle 
* dump函数用法 (处理文件)
``` python
test_list = ['a','b','c']
with open('test.txt', 'wb') as f:
    cPickle.dump(test_list, f)
```
* load 函数用法(处理文件)
``` python
with open('test.txt', 'rb') as f：
    test_list = cPickle.load(f)
    print test_list
>>> ['a','b','c']
```
* dumps 函数用法
```python
test_list = ['a', 'b', 'c']
obj = cPickle.dumps(test_list)
print type(obj)
>>> <type 'str'>
obj 以python专用的形式存储
```
* loads 函数用法
``` python
load_obj = cPickle.loads(obj)
print load_obj
>>> ['a', 'b', 'c']
```

#### json 格式
* dump函数用法 (处理文件)
``` python
test_list = ['a','b','c']
with open('test.txt', 'wb') as f:
    json.dump(test_list, f)
```
* load 函数用法(处理文件)
``` python
with open('test.txt', 'rb') as f：
    test_list = json.load(f)
    print test_list
>>> [u'a', u'b', u'c']
```
* dumps 函数用法
```python
test_list = ['a', 'b', 'c']
obj = json.dumps(test_list)
print type(obj)
>>> <type 'str'>
obj 以python专用的形式存储
```
* loads 函数用法
``` python
load_obj = json.loads(obj)
print load_obj
>>> [u'a', u'b', u'c']
```




