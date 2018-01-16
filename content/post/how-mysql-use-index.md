---
title: "How MySQL Uses Indexes"
tags: ["MySQL", "Index"]
date: 2018-01-15T14:52:59+08:00
#draft: true
---

Indexes are used to find rows with specific column values quickly. Without an index, MySQL must begin with the first row and then read through the entire table to find the relevant rows. The larger the table, the more this costs. If the table has an index for the columns in question, MySQL can quickly determine the position to seek to in the middle of the data file without having to look at all the data. This is much faster than reading every row sequentially.
```翻译:
索引被用来快速的找到特定的列值。如果没有索引，MySQL为了找到相关的数据必须从第一行开始读完整张表。这时候，表越大，所耗费的时间越长。如果这张表在所查询的这一列上有创建索引的话，MySQL能快速的决定要移动到数据文件的中间位置，而不必查看全部数据。这样肯定比按顺序逐行的读取要快很多。
```

Most MySQL indexes (`PRIMARY KEY`, `UNIQUE`, `INDEX`, and `FULLTEXT`) are stored in `B-trees`. Exceptions are that indexes on spatial data types use `R-trees`, and that MEMORY tables also support `hash` indexes.
```翻译:
MySQL的大部分索引(PRIMARY KEY, UNIQUE, INDEX, and FULLTEXT) 都存储在B-Tree中。例外情况是受空间限制的数据类型的索引使用R-tree存储，并且它的内存表支持hash索引。
```

MySQL uses indexes for these operations:

* To find the rows matching a `where` clause quickly.
```翻译：
快速的找到匹配where条件子句的行。
```

* To eliminate rows from consideration. If there is a choice between multiple indexes, MySQL normally uses the index that finds the smallest number of rows (the most selective index).
```翻译：
从考察的数据集中排除一些行。如果在多个索引之间进行选择，MySQL通常选择查询到最少行数的那个索引。
```

* If the table has a multiple-column index, any leftmost prefix of the index can be used by the optimizer to look up rows. For example, if you have a three-column index on (col1, col2, col3), you have indexed search capabilities on (col1), (col1, col2), and (col1, col2, col3). 
```翻译:
如果表中有一个多列索引，那么只有索引中的最左前缀可以被用来优化查询。例如，你有一个三列索引建在（col1, col2, col3）上，你只能在查询条件为(col1), (col1, col2), (col1, col2, col3)时可以使用索引优化查询。
```

*    To retrieve rows from other tables when performing joins. MySQL can use indexes on columns more efficiently if they are declared as the same type and size. In this context, VARCHAR and CHAR are considered the same if they are declared as the same size. For example, VARCHAR(10) and CHAR(10) are the same size, but VARCHAR(10) and CHAR(15) are not.

     For comparisons between nonbinary string columns, both columns should use the same character set. For example, comparing a utf8 column with a latin1 column precludes use of an index.

     Comparison of dissimilar columns (comparing a string column to a temporal or numeric column, for example) may prevent use of indexes if values cannot be compared directly without conversion. For a given value such as 1 in the numeric column, it might compare equal to any number of values in the string column such as '1', ' 1', '00001', or '01.e1'. This rules out use of any indexes for the string column.
    ```翻译:
    当执行join操作，从其他表中检索数据时。如果索引所在的列被申明成相同的类型和大小，MySQL能够更高效的使用索引。在这种情况下，VARCHAR 和 CHAR被认为是相同的，如果它们被申明成相同的大小。例如， VARCHAR(10) 和 CHAR(10) 是相同的，但是 VARCHAR(10) 和 CHAR(15) 是不同的。

    在比较非二进制字符串时，所有列必须使用相同的字符集。例如，将一个utf8列值和一个latin1列值比较，是无法使用索引的。

    比较不相似的列值，如果列值不能直接被比较而需要进行转换(例如:比较一个字符列和一个表示时间的列或者一个数字列)可能阻碍使用索引。
    ```

* To find the MIN() or MAX() value for a specific indexed column key_col. This is optimized by a preprocessor that checks whether you are using WHERE key_part_N = constant on all key parts that occur before key_col in the index. In this case, MySQL does a single key lookup for each MIN() or MAX() expression and replaces it with a constant. If all expressions are replaced with constants, the query returns at once. For example:
```SQL:
    SELECT MIN(key_part2), MAX(key_part2) FROM tab_name WHERE key_part1=10;

    查询一个指定索引列key_col上的最小和最大值。
```

* To sort or group a table if the sorting or grouping is done on a leftmost prefix of a usable index (for example, ORDER BY key_part1, key_part2). If all key parts are followed by DESC, the key is read in reverse order.
```翻译:
排序或者分组
```

* In some cases, a query can be optimized to retrieve values without consulting the data rows. (An index that provides all the necessary results for a query is called a covering index.) If a query uses from a table only columns that are included in some index, the selected values can be retrieved from the index tree for greater speed:
```SQL:
    SELECT　key_part3 FROM tab_name WHERE key_part1=1;

    哈哈哈
```

Indexes are less important for queries on small tables, or big tables where report queries process most or all of the rows. When a query needs to access most of the rows, reading sequentially is faster than working through an index. Sequential reads minimize disk seeks, even if not all the rows are needed for the query.
```翻译:
索引对于数据量很少的表，或者一张数据量很大的表但需要查询处理大部分行时，意义不大。当一个查询需要获取表中大部分行时，顺序的读取比使用索引更快。顺序的读取数据可以最小化磁盘寻找，即使不是所有的行都需要。
```
