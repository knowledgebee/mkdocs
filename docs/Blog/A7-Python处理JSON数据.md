---
title: Python 处理 JSON数据
date: 2022-09-14 13:04:11
tags:
    - Script
---

使用 Python 处理 JSON 格式数据

<!--more-->

调用软件 API 获取的数据为 JSON 格式的原始数据，不便于直接读取。提取原始中的信息还需要行计算转换为便于理解的信息，因此编写一个Python脚本来处理数据转换格式，并添加描述便于理解和处理。

Python 有着丰富的库函数，简单的语法结构，是一个非常适合编写脚本的语言。

Python 不能直接处理识别 JSON 格式的数据，当用 Python 来处理JSON数据时需要先引入 json 库，调用 JSON 函数将 JSON 格式的数据转换为 Python 字典格式的数据：

~~~Python
import json
~~~

为了能够方便的调用读取数据文件，还需要引入 sys 库。通过 sys 库来读取数据文件名称，以及读取文件和写入转换后的数据:

~~~Python
import os
~~~

打开通过 CLI (命令行) 输入的参数以只读的方式打开指定文件，并将文件内容保存在参量 `f` 中后关闭文件，将保存在参量 `f` 中的数据通过调用 JSON 库函数转换后保存在 `status_data` 中：

~~~Python
f = open(sys.argv[1], 'r')    # open(filename,'r') 为 sys 函数，打开文件 filename，sys.argv[a]同为sys函数，值为命令行参数的第a个
status_data = json.load(f)    # json.load() JSON 库函数，转换 JSON 格式数据为 Python 字典格式数据
f.close()                     # 关闭打开的文件 f
~~~

用大括号定义字典，向字典中写入数据：

~~~Python
in_and_out = {}                    # 创建字典 in_and_out
in_and_out["out"] = data_value     # 向字典 in_and_out 中写入键值对：键 out 值 data_value
~~~

用 len() 函数测量列表 list 中数据的个数：

~~~Python
length = len(status_data["stat"])  # 读取列表 status_data["stat"] 的长度并写入变量 length
~~~

获取字典中键的值：

~~~Python
data = status_data_in_list[i]      # 向变量 data 中写入键值对
data_value = data.get("value",0)   # 获取字典 data 中键为 value 的值，如果返回 value 的值，否则返回 0
~~~

在文件以写入方式打开后便可将内容写入文件，调用 print 函数写入，写入之后保存关闭文档：

~~~Python
f = open(sys.argv[1], 'w')         # 以写入方式打开文件 sys.argv[1] ，原有文件会被清空，如果文件不存在则创建文件
print(data_value,file=f)           # 将 data_value 的值写入文件 f 中，会自动换行
f.close()                          # 关闭文件
~~~

通过以上函数的调用已经可以对文件进行读写操作，并对 JSON 格式的文件进行处理。
