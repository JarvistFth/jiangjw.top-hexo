---
title: MYSQL - 索引
date: 2021-02-27 18:44:28
categories: 
- 数据库
tags: [数据库]
keywords: [数据库]

---
索引相关。
<!---more--->

## 索引底层数据结构类型

### B+树索引

底层是b+树。

### 哈希索引

innodb自适应hash索引，不能人工生成。

### FULLTEXT索引

基于分词器，计算查询关键词的相关度。

## 索引种类

### 普通索引

对普通索引建B+树存储，叶子结点都是主键；找到主键以后再根据主键找对应的tuple。

### 唯一索引

不允许重复；

### 主键索引

primary key， innodb默认必须带primary key；根据primary key

### 全文索引
对一个文档设定的fulltext index；通过查询关键字使用，避免LIKE查询。

### 复合索引
对多个数据列建立索引，遵循最左匹配原则。





