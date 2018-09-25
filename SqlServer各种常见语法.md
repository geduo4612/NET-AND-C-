打牢基础是十分关键。
首先是常见的表复制操作：

1 insert into T2 select * from T1  与 select * into T2 from T1的区别：

前者要求目标表Table2必须存在，由于目标表Table2已经存在，所以我们除了插入源表Table1的字段外，还可以插入常量。

后者要求目标表Table2不存在，因为在插入时会自动创建表Table2，并将Table1中指定字段数据复制到Table2中。

2 TRUNCATE table T1;然后再往T1插入数据时只能选用insert into T2 select * from T1 的方式，而且必须将列名全部显式的罗列出来才行，也就是说只能写成
  
  insert into T2 select column1，column2,column3... from T1 
