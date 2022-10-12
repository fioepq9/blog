---
title: "Python常用刷题函数"
date: 2022-10-10T23:38:11+08:00
draft: false
---

# 输入输出

```python
# 读取一行，去除头尾空格和换行，以空格划分读取为整数，最后放入list。
slice = list(map(int, input().strip().split()))

# 不断读取，直到EOF
def readUntilEof():
    try:
        while True:
            yield input()
    except:
        return
for ipt in readUntilEof():
    # do something

# 格式化字符串
"{:0>2d}".format(1) # 以0填充，右对齐2宽度
"{:0<2d}".format(1) # 以0填充，左对齐2宽度
"{:^10d}".format(1) # 居中10宽度
"{0.value}".format(myVal) # 输出myVal的成员value
"{:.2f}".format(3.14159)  # 保留2位小数
"{:+.2f}".format(3.14159) # 带符号保留2位小数
"{:b}".format(10)  # 输出2进制，1010
"{:d}".format(10)  # 输出10进制，10
"{:o}".format(10)  # 输出8进制，12
"{:x}".format(10)  # 输出16进制，a
"{:#x}".format(10) # 输出16进制，0xa
"{:#X}".format(10) # 输出16进制，0XA
```

# 内置函数

```python
# 进制
bin(number) # 返回number的2进制表示
oct(number) # 返回number的8进制表示
hex(number) # 返回number的16进制表示
# 数学
abs(number) # 返回number的绝对值
pow(x, y)   # 返回x的y次方
# 字符
chr(number) # 将ASCII值转换为char
ord(char)   # 将char转换为ASCII值
# 遍历
enumerate(itertor) # 返回索引，索引对应的元素
zip(iterable...)   # 同时遍历n个序列
# 语法糖
sum(number...) # 求和
max(number...) # 返回最大值
min(number...) # 返回最小值
range(start, stop, [step]) # 返回[start, stop)内以step为步长的序列
map(function, iterable)    # 对所有元素调用function并返回
filter(function, iterable) # 过滤掉不符合条件的元素
```

# 常用库

```python
import math
import collections
import heapq
import bisect
```

# 常用数据结构

```python
# 内置类型
list, tuple, dict, set
# collections库内结构
collections.deque
collections.Counter
collections.OrderedDict
```
