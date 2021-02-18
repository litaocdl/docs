## This is about python learning from tao
---



### Reference 
Python New Thumb: https://www.runoob.com/python/python-tutorial.html


### Python lib in Scientifics 
Scientifics computing lib
  * Pandas   
    Data Structure and tools 
  * Numpy  
    Arrays and Metrics . 
  * Scipy

Visualization lib  
  * Matplotlib . 
  * Seaborn . 

Machine Learning Algorithmic . 
  * Scikit-learning   
     Regression, classification  
  * StatsModels   
     Explorer data, perform staticitics test
  
---
## Data Types   
六种类型  

* 数字类型  
  int, float, bool,complex
  
  ```
   Input: type(28) type(0b111000) type(0o34) type(0x1c)  
   Output: class 'int'  
   二进制：0b开头，八进制：0o开头 十六进制： ox开头  
  ```
  
* 序列类型  
  有序的封装的类型  
  1. 字符串类型 str  
     索引  
     
    ```
      a=hello  
      H  e  l  l  o  
      0  1  2  3  4  
     -5 -4 -3 -2 -1 
    
    ```
    
    正索引0开头，负索引-1开头  a[0]==a[-5]  
    
    切片 slicing 
    ```
     [start:end:step] 包含start,不包含end  
     a[0:3:1]=Hel  
    ```
    
    成员测试 in, not in  
    
    长字符串：保持文本格式，用三个单引号'''或三个双引号"""括起来  
   

    
  2.字节序列 bytes  
  
  3.列表 list  
   可变序列类型   
   a=[1,2,3,4,5]  
   a=list('hello')  
  
  4.元祖 tuple  
  不可变序列类型  
  a=(1,2,3,4,5)  
  a=tuple('hello')  
  
* 集合  
  set 无序，可迭代，不能包含重复元素的容器类型  
  a={1,2,3,4,5}  
  a=set('hello')  ==> {'h','e','l','o'}  
  
* 字典  
  dict,可迭代，通过key访问value的容器类型  
  a={k1:1,k2:2,k3:3}  
  a=dict(((k1,1),(k2,2)))  

## 函数

```
 def funName(参数列表):
     函数体
     return 返回值
     
 def sum(num1=5,num2=2):
     return num1+num2
     
 可变参数 *var
 def sum(*num):
    i=0
    ##可变参数被封装成元祖变量
    for n in num:
       i += n
    return i
    
 可变参数封装为元祖调用
 
 sum(1,2,3,4,5)
 
 基于字典的可变参数 **var
 
 def sumValue(**var):
     i=0
     for k,v in var.items()
        i += v
     return i
```

函数：定义在模块中，类之外
嵌套函数：定义在函数中
方法：定义在类中
