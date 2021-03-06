---
title: 【高性能MySQL】- 第4章 Schema与数据类型优化
date: 2022-04-29 13:35:17
tags:
  - 阅读笔记
  - MySQL
  - 高性能MySQL
categories:
  - 数据库
  - MySQL
---

##  如何选择正确的数据类型？

+ 更小的通常更好。

一般情况下，应该尽量使用可以正确存储数据的最小数据类型。更小的数据类型通常更快，因为它们占用更少的磁盘、内存和 CPU 缓存，并且处理时需要的 CPU 周期也更少。

（确保没有低估需要存储的值得范围，因为在 schema 中得多个地方增加数据类型得范围是一个非常耗时和痛苦得操作。）

+ 简单就好。

简单数据类型得操作通常需要更少的 CPU 周期。

（例子：应该使用 MySQL 内建的类型而不是字符串来存储日期和时间；应该使用整型存储 IP 地址。）

+ 尽量避免 NULL。

很多表都包含可为 NULL 的列，即使应用程序并不需要保存 NULL 也是如此，这是因为可为 NULL 是列的默认属性。通常情况下最好指定列为 NOT NULL，除非真的需要存储 NULL 值。

通常把可为 NULL 的列改为 NOT NULL 带来的性能提升比较小，所以调优时没有必要首先在现有 schema 中查找并修改掉这种情况，除非确定这会导致问题。但是，如果计划在列上建立索引，就应该尽量避免设计为可为 NULL 的列。

### 使用 NULL 的缺点

+ 如果查询中包含可为 NULL 的列，对 MySQL 来说更难优化，因为可为 NULL 的列使得索引、索引统计和值比较都更为复杂。
+ 可为 NULL 的列会使用更多的存储空间，在 MySQL 里也需要特殊处理。
+ 当可为 NULL 的列被索引时，每个索引记录需要一个额外的字节，在 MyISAM 里甚至还可能导致固定大小的索引变成可变大小的索引。

## 数字类型

### 整数（whole number）

+ TINYINT：8位
+ SMALLINT：16位
+ MEDIUMINT：24位
+ INT：32位
+ BIGINT：64位

MySQL 使用 BIGINT 作为内部整数计算的类型。

整数类型有可选的 UNSIGNED 属性，表示不允许负值，这大致可以使正数的上限提高一倍。

有符号和无符号类型使用相同的存储空间，并具有相同的性能。

MySQL 可以为整数类型指定宽度，例如 INT(11)，这不会限制值得合法范围，只是规定了 MySQL 的一些交互工具用来显示字符的个数。

### 实数（real number）

+ DECIMAL：最多65个数字。用于存储精确的小数，支持精确计算。可以指定小数点前后所允许的最大位数。

+ FLOAT：占用4个字节。
+ DOUBLE：占用8个字节。

MySQL 使用 DOUBLE 作为内部浮点计算的类型。（即计算时，DECIMAL 和 FLOAT 都会转换为 DOUBLE。）

因为需要额外的空间和计算开销，所以应该尽量只在对小数进行精确计算时才使用 DECIMAL。（但在数据量比较大的时候，可以考虑使用 BIGINT 代替 DECIMAL。这样可以同时避免浮点存储计算不精确和 DECIMAL 精确计算代价高的问题。）

## 字符串类型

#### VARCHAR 类型

VARCHAR 类型用于存储可变长字符串，是最常见的字符串数据类型。它比定长类型更节省空间，因为它仅使用必要的空间。（例外情况：MySQL 表使用 ROW_FORMAT=FIXED 创建，这代表每一行是定长的，对于 VARCHAR 会很浪费空间。）

##### VARCHAR 的特点

+ VARCHAR 需要使用额外空间记录字符串的长度。

  + 如果列的最大长度小于或等于 255 字节，则只使用 1 个字节。	

  + 如果列的最大长度大于 255 字节，则使用 2 个字节。

+ 在 UPDATE 时可能使行变得比原来更长，这就导致需要做额外的工作。

##### 什么时候使用 VARCHAR ？

1. 字符串列的最大长度比平均长度大很多。
2. 列的更新很少，所以碎片不是问题。
3. 使用了像 UTF-8 这样复杂的字符集，每个字符都使用不同的字节数进行存储。

##### 使用 VARCHAR(5) 和 VARCHAR(200) 存储 'hello' 的空间开销是一样的，那么使用更短的列有什么优势吗？

虽然这两种类型存储 'hello' 在磁盘上的空间开销一致，但因为 MySQL 通常会分配固定大小的内存块来保存内部值（即使用定义的长度），更长的列会消耗更多的内存，这导致使用内存临时表进行排序或操作时会特别糟糕（在利用磁盘临时表进行排序时也同样糟糕）。

最好的策略是只分配真正需要的空间。

#### CHAR 类型

CHAR 类型是定长的，MySQL 总是根据定义的字符串长度分配足够的空间。

##### CHAR 的特点

+ 当存储 CHAR 值时，MySQL 会删除所有的末尾空格。
+ CHAR 值会根据需要采用空格进行填充以方便比较。

##### 什么时候使用 CHAR ?

+ 存储很短的字符串，或者所有值都接近同一个长度。
+ 经常需要 UPDATE 的数据。

### BINARY 类型和 VARBINARY 类型

这两种与 CHAR 类型和 VARCHAR 类型类似，只是存储的是二进制字符串。

二进制比较比字符比较简单很多，所以更快。

MySQL 会根据需要使用 \0 填充 BINARY 类型。

### BLOB 类型和 TEXT 类型

BLOB 和 TEXT 都是为存储很大的数据而设计的字符串数据类型，分别采用二进制和字符形式存储。

二进制类型：TINYBLOB，SMALLBLOB，BLOB，MEDIUMBLOB，LONGBLOB。

字符类型：TINYTEXT，SMALLTEXT，TEXT，MEDIUMTEXT，LONGTEXT。

BLOB 是 SMALLBLOB 的同义词，TEXT 是 SMALLTEXT 的同义词。

#### BLOB 和 TEXT 的特点

+ MySQL 把每个 BLOB 和 TEXT 值当作一个独立的对象处理。存储引擎在存储时通常会做特殊处理。当 BLOB 和 TEXT 值太大时，InnoDB 会使用专门的「外部」存储区域来进行存储，此时每个值在行内需要 1 ~ 4 个字节存储一个指针，然后再外部存储区域存储实际的值。
+ MySQL 对 BLOB 和 TEXT 列进行排序与其他类型是不同的：它只对每个列的最前 max_sort_length 字节而不是整个字符串做排序。如果只需要排序前面一小部分字符，则可以减少 max_sort_length 的配置，或者使用 ORDER BY SUBSRING(column, length)
+ MySQL 不能将 BLOB 和 TEXT 列全部长度的字符串进行索引，也不能使用这些索引消除排序。

#### BLOB 类型和 TEXT 类型的不同？

BLOB 类型存储的是二进制数据，没有排序规则或字符集，而 TEXT 类型有字符集和排序规则。

## 使用 ENUM 代替字符串类型

MySQL 在存储枚举时非常紧凑，会根据列表值的数量压缩到一个或者两个字节中。

MySQL 在内部会将每个值在列表中的位置保存为整数，并且在表的 *.frm* 文件中保存「数字-字符串」的映射关系的「查找表」。

ENUM 按照内部存储的整数进行排序而不是定义的字符串。

ENUM 最不好的地方是，字符串列表固定，添加或删除字符串必须使用 ALTER TABLE。

## 日期和时间类型

### DATETIME

+ 时间范围：1001年 - 9999年，精度为秒。
+ 存储格式：把日期和时间封装到格式为 YYYYMMDDHHMMSS 的整数中，与时区无关。
+ 空间开销：8字节。

默认情况下，MySQL 以一种可排序的、无歧义的格式显示 DATETIME 值，例如「2008-01-16 22:37:08」（ANSI标准）。

### TIMESTAMP

+ 时间范围：1970年 - 2038年，精度为秒。
+ 存储格式：从 1970 年 1 月 1 日午夜（格林尼治标准时间）以来的秒数，与 UNIX 时间戳相同。与时区有关。
+ 空间开销：4字节。

默认情况下，

+ 如果插入时没有指定第一个 TIMESTAMP 列的值，MySQL 则设置这个列的值为当前时间。
+ 在插入一行记录时，MySQL 默认也会更新第一个 TIMESTAMP 列的值（除非在 UPDATE 语句中明确指定了值）。
+ TIMESTAMP 列默认为 NOT NULL，这也和其他的数据类型不一样。

## 位数据类型

### BIT







