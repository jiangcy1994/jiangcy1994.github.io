---
layout: default
title:  "三方面提升pandas处理大型CSV文件性能"
date:   2018-04-16 13:00:29
categories: pandas
---

# 三方面提升pandas处理大型CSV文件性能

几年来，随着数据处理的发展，需要处理的数据量日益增长，如何处理巨大的数据文件成为了一个比较重要的问题。由于大多数数据可以转化为CSV结构的数据，本博客根据[0.22.0版pandas的技术文档](http://pandas.pydata.org/pandas-docs/version/0.22/)，提出了可能提升pandas在面临大文件读取问题时，以及有限硬件条件时的，提升处理性能的解决方案。

## 1. 处理器

基于pandas文档中pandas.read_csv函数中的参数engine，文档提出了如下的说明：

> engine : {'c', 'python'}  
> &emsp;&emsp;Parser engine to use. The C engine is faster while the python engine is currently more feature-complete.

因而，我们可以利用c处理器相对较快的优点，对数据进行预先的处理，并将结果重新转存为新的CSV文件。此时，新的CSV文件一般较先前的CSV文件小，从而达到了提升pandas处理大型csv性能的目的。

适用范围：对数据进行归纳、总结及相关处理。通常会表现出数据文件大小越处理越小的情况。

## 2. 文件分块

基于pandas文档中pandas.read_csv函数中的参数skiprows和nrows，文档提出了如下的说明：

> skiprows : list-like or integer, default None  
> &emsp;&emsp;Line numbers to skip (0-indexed) or number of lines to skip (int) at the start of the file.  
> &emsp;&emsp;If callable, the callable function will be evaluated against the row indices, returning True if the row should be skipped and False otherwise:  
> nrows : int, default None  
> &emsp;&emsp;Number of rows of file to read. Useful for reading pieces of large files.

通过设置跳过文件开头行数\(skiprows\)和读取文件行数\(nrows\)，我们可以实现对文件内部选定范围内的数据读取，及分块读取。从而，我们可以减少出现一次性读入过多文件信息的情况。

适用范围：数据处理为对文件块内数据进行处理，文件块间数据处理较为稀少。

## 3. 文件解析

基于pandas文档中pandas.read_csv函数中的参数na_filter，文档提出了如下的说明：

> na_filter : boolean, default True  
> &emsp;&emsp;Detect missing value markers (empty strings and the value of na_values). In data without any NAs, passing na_filter=False can improve the performance of reading a large file.

在读取大文件时，pandas会默认将[这些数值](http://pandas.pydata.org/pandas-docs/version/0.22/io.html#io-navaluesconst)认为是缺失值，减少文件解析过程中的过程可以提升读取的性能

适用范围：无需使用文件解析中的过滤步骤的数据文件。
