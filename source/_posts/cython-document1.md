---
title: cython-document1
date: 2017-02-03 14:48:10
tags: [cython]
categories: cython文档阅读
---
## cython -Extention types
### 一般的python class
``` python
class MathFunction(object):
    def __init__(self, name, operator):
        self.name = name
        self.operator = operator
    def __call__(self, *operands):
        return self.operator(*operands)
```
普通的python 是用dictionary 去存储class中对象和方法，对于cython可以采用c中的结构体去更加高效的存储
### cython class
下面是一个简单的cython类的定义
``` python
cdef class Function:
    cpdef double evaluate(self, double x) except *:
        return 0
```
我们再定义一个类继承上面的类
``` python
cdef class SinOfSquareFunction(Function):
    cpdef double evaluate(self, double x) except *:
        return sin(x**2)
```
cpdef 表示python 外部可调用此方法，注意对于cpdef 定义子类会完全重载父类中的方法

<!-- more -->
### 简单的例子
* cython 格式
``` python
def integrate(Function f, double a, double b, int N):
    cdef int i
    cdef double s, dx
    if f is None:
        raise ValueError("f cannot be None")
    s = 0
    dx = (b-a)/N
    for i in range(N):
        s += f.evaluate(a+i*dx)
    return s * dx
print(integrate(SinOfSquareFunction(), 0, 1, 10000))
```
这是一段简单的cython的代码，我们将通过这个来感受cython带来的速度提升
* python 格式
``` python
>>> import integrate
>>> class MyPolynomial(integrate.Function):
...     def evaluate(self, x):
...         return 2*x*x + 3*x - 10
...
>>> integrate(MyPolynomial(), 0, 1, 10000)
```
我们这里直接的生成一个python的类去继承Function这个类，并重载Function的方法
* 速度差异
这种python格式的class 比cython 格式要慢20倍，但是比原生的python还是要快10倍
* 注意事项
    * 首先Function中必须要声明evaluate方法，否则SinOfSquareFunction中会默认去调python版本的evaluate
    * 参数f 必须要声明类型即Function
    * 参数必须做检查是否为None
* @cython.nonecheck(False) 可做None 检查但是会消耗一定时间
``` python
import cython
# Turn off nonecheck locally for the function
@cython.nonecheck(False)
def func():
    cdef MyClass obj = None
    try:
        # Turn nonecheck on again for a block
        with cython.nonecheck(True):
            print obj.myfunc() # Raises exception
    except AttributeError:
        pass
    print obj.myfunc() # Hope for a crash!
```
### cdef class 属性注意事项
* 所有属性编译前必须预先声明
* 属性必须是cython 所允许的
* Properties 可以被声明在python 空间中
* 实例
``` python
cdef class WaveFunction(Function):
    # Not available in Python-space:
    cdef double offset
    # Available in Python-space:
    cdef public double freq
    # Available in Python-space:
    @property
    def period(self):
        return 1.0 / self.freq
    @period.setter
    def period(self, value):
        self.freq = 1.0 / value
```










 

