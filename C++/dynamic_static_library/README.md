# linux下动态库与静态库的生成与使用

## 定义

静态与动态的区分主要是在编译系统中的**链接**部分

- 静态库在程序链接的时候会自动的链接到程序里，所以一旦编译完成，静态库就不需要了，静态库以.a结尾。
-  动态库在编译时不会被连接到目标代码中，而是在程序运行时才被载入，动态库以.so结尾。 

静态库相对于动态库来说更高的效率，但是消耗更多的空间。

## 生成

~~~c++
//hello.h
#ifndef _HELLO_H
#define _HELLO_H
void hello(const char* str);
#endif

//hello.c
#include <stdio.h>  
void hello(const char* name){  
    printf("hello %s! \n",name);  
}

//main.c
#include "hello.h"  
int main(){  
	hello("everyone");  
	return 0;  
} 
~~~

### 静态库

- 生成静态库，将hello.c编译成hello.o文件，然后利用ar工具将hello.o文件打包成.a文件。注意的是静态库命名规范为lib[library_name].a。

  ~~~
  gcc - c hello.c
  ar crv libhello.a hello.o
  ~~~

- 使用静态库，需要用-L参数指定静态库位置(不在默认文件夹时)，-l参数指定静态库名(不加lib前缀和.a后缀)

  ~~~
  gcc -o hello main.c -L. -lhello
  ~~~

### 动态库

- 生成动态库有两种方法，第一种，将hello.c编译成hello.o文件(动态库需加参数-fPIC，目的是为在多个程序间共享)，然后生成动态库。第二种，将上述步骤合并为一条指令。

  ~~~
  //第一种
  gcc -fPIC hello.c
  gcc -shared -o libhello.so hello.o
  //第二种
  gcc -shared -fPIC -o libhello.so hello.c
  ~~~

- 动态库的使用与静态库一致

  ~~~
  gcc -o hello main.c -L. -lhello
  ~~~

  PS：如果出现动态库或静态库无法找到的错误，可将其移至默认搜索目录下，/usr/lib或/lib, 并更新 ldconfig

