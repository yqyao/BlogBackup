---
title: cython-document2
date: 2017-02-03 16:29:27
tags: [cython]
categories: cython文档阅读
---
## Using C libraries
为了能更高效的利用c的library，cython提供了一套API。用户可以兼顾python开发的高效性与c性能的高效性。下面以cython调用c的queue为例简单介绍。
### C的queue 头文件(queue.h)
``` cpp
/* file: queue.h */

typedef struct _Queue Queue;
typedef void *QueueValue;

Queue *queue_new(void);
void queue_free(Queue *queue);

int queue_push_head(Queue *queue, QueueValue data);
QueueValue queue_pop_head(Queue *queue);
QueueValue queue_peek_head(Queue *queue);

int queue_push_tail(Queue *queue, QueueValue data);
QueueValue queue_pop_tail(Queue *queue);
QueueValue queue_peek_tail(Queue *queue);

int queue_is_empty(Queue *queue);
```
### 重定义C的头文件（cqueue.pxd）
<!-- more -->
``` cpp
# file: cqueue.pxd

cdef extern from "libcalg/queue.h":
    ctypedef struct Queue:
        pass
    ctypedef void* QueueValue

    Queue* queue_new()
    void queue_free(Queue* queue)

    int queue_push_head(Queue* queue, QueueValue data)
    QueueValue  queue_pop_head(Queue* queue)
    QueueValue queue_peek_head(Queue* queue)

    int queue_push_tail(Queue* queue, QueueValue data)
    QueueValue queue_pop_tail(Queue* queue)
    QueueValue queue_peek_tail(Queue* queue)

    bint queue_is_empty(Queue* queue)
```
对于cython头文件的格式以.pyd的格式存在，对于每一个所用的c library 建立一个.pxd文件是一个非常好的习惯

### 队列class 
``` python
# file: queue.pyx
cimport cqueue

cdef class Queue:
    cdef cqueue.Queue* _c_queue
    def __cinit__(self):
        self._c_queue = cqueue.queue_new()
```
注意如果用了cdef 定义class时，相应了要用__cinit__()。
### 内存分配
在调用queue_new() 时肯定会分配内存，如果内存不足就会出现错误，因此上面的queue.pyx需要改进,在内存不足时需要报内存错误。
``` python
cimport cqueue

cdef class Queue:
    cdef cqueue.Queue* _c_queue
    def __cinit__(self):
        self._c_queue = cqueue.queue_new()
        if self._c_queue is NULL:
            raise MemoryError()
```
同样的在内存释放时也需要做检查
``` python
def __dealloc__(self):
    if self._c_queue is not NULL:
        cqueue.queue_free(self._c_queue)
```

### 编译和链接
* setup.py （不需要加入include）
``` python
from distutils.core import setup
from distutils.extension import Extension
from Cython.Build import cythonize

setup(
    ext_modules = cythonize([Extension("queue", ["queue.pyx"])])
)
```
* 使用c的library 需要加入include
``` python
ext_modules = cythonize([
    Extension("queue", ["queue.pyx"],
              libraries=["calg"])
    ])
```
* 运行时指定 include
``` bash
CFLAGS="-I/usr/local/otherdir/calg/include"  \
LDFLAGS="-L/usr/local/otherdir/calg/lib"     \
    python setup.py build_ext -i
```
### 映射方法
* append()
``` python
cdef append(self, int value):
    if not cqueue.queue_push_tail(self._c_queue, <void*>value):
        raise MemoryError()
```
* extend()
``` python
cdef extend(self, int* values, size_t count):
    """Append all ints to the queue.
    """
    cdef size_t i
    for i in range(count):
        if not cqueue.queue_push_tail(
                self._c_queue, <void*>values[i]):
            raise MemoryError()
```
* peek() and pop()
``` python
cdef int peek(self):
    return <int>cqueue.queue_peek_head(self._c_queue)

cdef int pop(self):
    return <int>cqueue.queue_pop_head(self._c_queue)
```
注意在append 和 extend 方法中同样也要做None 类型检查，原因同之前

### 队列为空问题
在c中队列为空，peek会返回一个空指针，对应cython中是0，如果队列的元素刚好也为0这样我们就无法分辨，因此我们希望返回的是一个异常。因此第二代版本的peek() 如下
``` python
cdef int peek(self) except? -1:
    value = <int>cqueue.queue_peek_head(self._c_queue)
    if value == 0:
        # this may mean that the queue is empty, or
        # that it happens to contain a 0 value
        if cqueue.queue_is_empty(self._c_queue):
            raise IndexError("Queue is empty")
    return value
```
相应的我们也增加了pop()的第二代版本
``` python
cdef int pop(self) except? -1:
    if cqueue.queue_is_empty(self._c_queue):
        raise IndexError("Queue is empty")
    return <int>cqueue.queue_pop_head(self._c_queue)
```


