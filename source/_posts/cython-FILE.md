---
title: cython_FILE
date: 2017-02-13 14:31:35
tags: [cython]
categories: cython学习
---
## cython FIlE 用法
### C中FILE的用法
* 定义文件指针变量
```cpp
FILE *fp;
```
* 打开操作的文件
```cpp
fp = fopen('test.txt','r'); //只读
fp = fopen('test.txt','rb'); //以二进制形式，只读
fp = fopen('test.txt','w'); //只写
fp = fopen('test.txt','wb'); //以二进制形式，只写
fp = fopen('test.txt','r+'); // 可读可写，指针指向文件头
fp = fopen('test.txt','rb+'); //以二进制形式
fp = fopen('test.txt','a+'); //可读可写，文件指针指向文件尾，即追加模式添加
fp = fopen('test.txt','ab+'); //以二进制形式
```
* 设置文件读写指针位置
``` cpp
//将文件读/写指针移到距文件头100字节处     
fseek( fp,  100L, SEEK_SET );
//将文件读/写指针从当前位置向文件尾方向移50字节  
fseek( fp,   50L, SEEK_CUR );                          
//将文件读/写指针从当前位置向文件头方向移50字节  
fseek( fp,  -50L, SEEK_CUR );  
//将文件读/写指针从文件尾回移100字节  
fseek( fp, -100L, SEEK_END );
```

**对文件进行读写操作**

* 格式：字节变量=fgetc( FILE *fp );
    * fp 这是个文件指针，它指出要从中读取字符的文件
    * 实例
```cpp
    //从文件中读取1个字节给ch     
      ch =fgetc( fp );  
```
<!-- more -->
* 格式：fputc( char *str, FILE *fp );
    * str：指出要写到文件中去的字符串。
    * fp：这是个文件指针，指出字符串要写入其中的文件。
    * 实例
```cpp
    //将ch(单字节)值写入文件     
      fputc( ch, fp );
```
* 格式：fputs/fgets(char *str, int n, FILE *fp );
    * str：接收字符串的内存地址，可以是数组名，也可以是指针。
    *   n： 指出要读取字符的个数。
    *  fp：这是个文件指针，指出要从中读取字符的文件
    *  实例
```cpp                       
    //从文件中读取字符串(5个字符)给str     
      fgets( str, 5, fp );   
    //将str字符串写入文件  
      fputs( str, fp );  
```
* 格式：fread/fwrite(void *buffer, unsigned size, unsigned count, FILE *fp );
    * buffer：这是一个void型指针，指出要将读入h或写入数据存放在其中的存储区首地址。
    * size：指出一个数据块的字节数，即一个数据块的大小尺寸。
    * count：指出一次读入多少个数据块（sife）。
    * fp：这是个文件指针，指出要从其中读出数据的文件。
    * 实例
```cpp
    int var[5];
    //从文件中读取5个指定字节长度数据给指定类型变量数组var     
      fread( var, sizeof(var[0]), 5, fp );                        
    //将指定类型变量数组var的前5个元素写入文件  
      fwrite( var, sizeof(var[0]), 5, fp );
```
**其他函数**
* fprintf() 和fscanf()
```cpp
//用法同printf
fprintf(fp,"%2d%s",4,"Hahaha")
// 从流中按格式读取
fscanf(fp,"%d%d" ,&x,&y)
```
* feof() 和 ferror()
```cpp
// 判断是否到文件尾
if(feof(fp))printf("已到文件尾");
//返回流最近的错误代码
printf("%d",ferror(fp)); 
```
* rewind() 和remove()
```cpp
//把当前的读写位置回到文件开始
rewind(fp); 
// 删除文件
remove("test.txt"); 
```
**简单的一个实例**
lib_c 这个文件前4个字节表示length，后面存储的是一个二维数组，维度为length*128，存储的是浮点数
```cpp
#include<stdio.h>
#include<stdlib.h>
int main()
{
    FILE *fp;
    int length;
    char *file_path;
    int file_size;
    float *array;
    file_path = "lib_c";
    fp = fopen(file_path, "rb");
    fread(&length, sizeof(int), 1, fp);
    printf("%d\n", length);
    file_size = length*128;
    array = (float*)malloc(sizeof(float)*file_size);
    fread(array, 4, file_size, fp);
    printf("%f\n", array[0]);
    fclose(fp);
    free(array);
    return 0;
}
```

### cython 中使用
``` python
from libc.stdib cimport*
from libc.stdio cimport*
def test():
    cdef:
        FILE *p
        int length
        float *array
        str file_path
    file_path = 'lib_c'
    fp = fopen(file_path, "rb");
    fread(&length, sizeof(int), 1, fp);
    file_size = length*128;
    array = <float*>malloc(sizeof(float)*file_size);
    fread(array, 4, file_size, fp);
    fclose(fp);
    free(array);
```
这里就得到了一个c中的二维数组，如果调用底层c的模块需要传入c中的数组，这里可以这么使用
