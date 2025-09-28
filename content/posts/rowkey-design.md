+++
title = "RowKey 设计规约"
date = 2020-09-25T00:00:00Z
tags = ["HBase"]
categories = ["notes"]
+++

# RowKey 尽量简短

Data Block 是 HBase 中文件读取的最小单元。Data Block中主要存储用户的KeyValue数据，而KeyValue结构是HBase存储的核心。HBase中所有数据都是以KeyValue结构存储在HBase中。

KeyValue由4个部分构成，分别为Key Length、Value Length、Key和Value。其中，Key Length和Value Length是两个固定长度的数值，Value是用户写入的实际数据，Key是一个复合结构，由多个部分构成：Rowkey、Column Family、Column Qualif ier、TimeStamp以及KeyType。其中，KeyType有四种类型，分别是Put、Delete、DeleteColumn和DeleteFamily。

HBase中数据在最底层是以KeyValue的形式存储的，其中Key是一个比较复杂的复合结构。

这也是HBase系统在表结构设计时经常强调Rowkey、Column Family以及ColumnQualif ier尽可能设置短的根本原因。列族一般情况下设置为 1 个。

——《HBase 原理与实战》

# 等长

类似桶排序规则。

# ID 在前

\<user_id\>\<业务字段\>



# 优先级排序

高频查询的越靠前

