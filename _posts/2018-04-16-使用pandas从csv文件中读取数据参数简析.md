---
layout: default
title:  "使用pandas从csv文件中读取数据参数简析"
date:   2018-04-16 13:00:29
categories: pandas
---

# 使用pandas从csv文件中读取数据参数简析

pandas从csv文件中提取数据一般使用read_csv函数，该函数会返回一个DataFrame或者一个TextFileReader

在本篇博客中，对读取函数参数中的字段名的默认值和可用值进行了归纳。因为可以通过字段名轻易的推导出字段含义，所以对字段名所代表的意思不做解析

## 基础参数

| 字段名             | 默认值 | 可用值  |
| ------------------ | ------ | ------- |
| filepath_or_buffer |        | Multi   |
| sep                | ','    | str     |
| delimiter          | None   | str     |
| delim_whitespace   | False  | boolean |

## 行列名

| 字段名           | 默认值  | 可用值                   | 备注               |
| ---------------- | ------- | ------------------------ | ------------------ |
| header           | 'infer' | int or list of ints      |                    |
| names            | None    | array-like               |                    |
| index_col        | None    | int or sequence or False |                    |
| usecols          | None    | array-like or callable   |                    |
| as_recarray      | False   | boolean                  | 在0.18.2中已被弃用 |
| squeeze          | False   | boolean                  |                    |
| prefix           | None    | str                      |                    |
| mangle_dupe_cols | True    | boolean                  |                    |

## 一般解析设置

| 字段名           | 默认值 | 可用值                     | 备注                               |
| ---------------- | ------ | -------------------------- | ---------------------------------- |
| dtpye            | None   | ype name or dict of column | 在0.20.0后增加对python解析器的支持 |
| engine           |        | {'c', 'python'}            |                                    |
| converters       | None   | dict                       |                                    |
| true_values      | None   | list                       |                                    |
| false_values     | None   | list                       |                                    |
| skipinitialspace | False  | boolean                    |                                    |
| skiprows         | None   | list-like or integer       |                                    |
| skipfooter       | 0      | int                        |                                    |
| skip_footer      | 0      | int                        | 在0.19.0中已被弃用                 |
| nrows            | None   | int                        | 大文件                             |
| low_memory       | True   | boolean                    | 仅对c解析器有效                    |
| buffer_line      | None   | int                        | 在0.19.0中已被弃用                 |
| compact_ints     | False  | boolean                    | 在0.19.0中已被弃用                 |
| use_unsinged     | False  | boolean                    | 在0.18.2中已被弃用                 |
| memory_map       | False  | boolean                    | 大文件                             |

## 空缺值设置

| 字段名           | 默认值 | 可用值                          | 备注   |
| ---------------- | ------ | ------------------------------- | ------ |
| na_values        | None   | scalar, str, list-like, or dict |        |
| keep_default_na  | True   | boolean                         |        |
| na_filter        | True   | boolean                         | 大文件 |
| verbose          | False  | boolean                         |        |
| skip_blank_lines | True   | boolean                         |        |

## 日期时间处理

| 字段名                | 默认值 | 可用值                                                    |
| --------------------- | ------ | --------------------------------------------------------- |
| parse_dates           | None   | boolean or list of ints or names or list of lists or dict |
| infer_datetime_format | False  | boolean                                                   |
| keep_date_col         | False  | boolean                                                   |
| date_parser           | None   | function                                                  |
| dayfirst              | False  | boolean                                                   |

## 遍历

| 字段名    | 默认值 | 可用值  |
| --------- | ------ | ------- |
| iterator  | False  | boolean |
| chunksize | None   | int     |

## 引号、压缩、文件格式

| 字段名          | 默认值  | 可用值                                      | 备注                              |
| --------------- | ------- | ------------------------------------------- | --------------------------------- |
| compression     | 'infer' | {'infer', 'gzip', 'bz2', 'zip', 'xz', None} | 在0.18.1中增加对zip和xz压缩的支持 |
| thousands       | None    | str                                         |                                   |
| decimal         | '.'     | str                                         |                                   |
| float_precision | None    | string                                      |                                   |
| lineterminator  | None    | str (length 1)                              | 仅对c解析器有效                   |
| quotechar       |         | str (length 1)                              |                                   |
| quoting         | 0       | int or csv.QUOTE_* instance                 |                                   |
| doublequote     | True    | boolean                                     |                                   |
| escapechar      | None    | str (length 1)                              |                                   |
| comment         | None    | str                                         |                                   |
| encoding        | None    | str                                         |                                   |
| dialect         | None    | str or csv.Dialect                          |                                   |
| tupleize_cols   | False   | boolean                                     | 在0.21.0中已被弃用                |

## 错误处理

| 字段名          | 默认值 | 可用值  |
| --------------- | ------ | ------- |
| error_bad_lines | True   | boolean |
| warn_bad_lines  | True   | boolean |


本篇博客基于pandas 0.22.0版本的文档