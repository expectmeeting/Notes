# 程序调试

## 1、段错误

发生段错误的原因：1、使用野指针；2、试图修改字符串常量的内容；3、数组越界访问

调试方法：

~~~
//首先，取消对吐核大小的限制
# ulimit -c unlimited 
//然后，重新执行出差程序，并找到core文件
# ./test
//采用gdb调试，找到错误位置
# gdb ./test core2344
~~~





## 常见错误

- 隐式声明与内建函数  -> 没有包含所使用函数的头文件
- 程序中有游离的‘\357’ ->程序中包含中文标点