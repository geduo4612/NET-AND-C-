因为数据库中存在几张大表（超过百万行），影响性能，所以才生分区的需求。该片主要记录操作过程中出现的一些问题。

首先明确本次操作的数据库版本，语法```select @@version```

> Microsoft SQL Server 2012 - 11.0.2100.60 (X64) 

要将一个原有的普通表改造成为分区表，步骤分为5步：

　　（1）创建数据库**文件组**

　　（2）创建数据库**文件**

　　（3）创建**分区函数**

　　（4）创建**分区方案**

　　（5）改造普通表为**分区表**
  
  具体的步骤可以参考[Sql Server系列：分区表操作](https://www.cnblogs.com/libingql/p/4087598.html)。
  
  遇到的问题主要是：
  
  （1）这篇[文章](http://www.cnblogs.com/sienpower/archive/2011/12/31/2308741.html) 中【定义分区构架】的第一幅图中，分区1和分区2 可以放在文件组1中，
     实际遇到“分区函数的分区多于分区架构指定的文件组”的错误。也就是说，分区函数的**分区边界**会产生**n+1**个分区，n表示分区边界，例如：
     
>CREATE PARTITION FUNCTION PF_YeeSkyGo_HotelPrice_DateRange(datetime)  AS RANGE RIGHT FOR VALUES (N'2018-07-01 00:00:00.000',  N'2018-09-01 00:00:00.000')
  
中分区边界为：N'2018-07-01 00:00:00.000',  N'2018-09-01 00:00:00.000',会产生3个分区。那么在创造分区架构的时候，指定的文件组也必须是3个。
  
（2）普通表变为分区表的注意点：     
>这个表拥有一般普通表的特性——有主键，同时这个主键还是聚集索引。 分区表是以某个字段为分区条件，所以，除了这个字段以外的其他字段，是不能创建聚集索引的。因此，要想将普通表转换成分区表，就必须要先删除聚集索引，然后再创建一个新的聚集索引，在该聚集索引中使用分区方案。在SQL Server中，如果一个字段既是主键又是聚集索引时，并不能仅仅删除聚集索引。因此，我们只能将整个主键删除，然后重新创建一个主键，只是在创建主键时，不将其设为聚集索引.
 
 
 完整的事例如下：
 ==============
 
```SQL
USE [YeeSkyGo]
GO

alter database [YeeSkyGo] add filegroup FG01_HotelPrice;  --增加文件组
alter database [YeeSkyGo] add filegroup FG02_HotelPrice;  --增加文件组
alter database [YeeSkyGo] add filegroup FG03_HotelPrice;  --增加文件组
alter database [YeeSkyGo] add filegroup FG04_HotelPrice;  --增加文件组
alter database [YeeSkyGo] add filegroup FG05_HotelPrice;  --增加文件组
alter database [YeeSkyGo] add filegroup FG06_HotelPrice;  --增加文件组
alter database [YeeSkyGo] add filegroup FG07_HotelPrice;  --增加文件组
alter database [YeeSkyGo] add filegroup FG08_HotelPrice;  --增加文件组
alter database [YeeSkyGo] add filegroup FG09_HotelPrice;  --增加文件组
alter database [YeeSkyGo] add filegroup FG10_HotelPrice;  --增加文件组
alter database [YeeSkyGo] add filegroup FG11_HotelPrice;  --增加文件组

go

alter database  [YeeSkyGo] add file   --增加文件
(
    NAME=F01_HotelPrice,
    FILENAME='d:\db\data\F01_HotelPrice.ndf',
    SIZE=1MB,
    FILEGROWTH=10%
)
TO filegroup FG01_HotelPrice;

alter database  [YeeSkyGo] add file   --增加文件
(
    NAME=F02_HotelPrice,
    FILENAME='d:\db\data\F02_HotelPrice.ndf',
    SIZE=1MB,
    FILEGROWTH=10%
)
TO filegroup FG02_HotelPrice;

alter database  [YeeSkyGo] add file   --增加文件
(
    NAME=F03_HotelPrice,
    FILENAME='d:\db\data\F03_HotelPrice.ndf',
    SIZE=1MB,
    FILEGROWTH=10%
)
TO filegroup FG03_HotelPrice;

alter database  [YeeSkyGo] add file   --增加文件
(
    NAME=F04_HotelPrice,
    FILENAME='d:\db\data\F04_HotelPrice.ndf',
    SIZE=1MB,
    FILEGROWTH=10%
)
TO filegroup FG04_HotelPrice;

alter database  [YeeSkyGo] add file   --增加文件
(
    NAME=F05_HotelPrice,
    FILENAME='d:\db\data\F05_HotelPrice.ndf',
    SIZE=1MB,
    FILEGROWTH=10%
)
TO filegroup FG05_HotelPrice;

alter database  [YeeSkyGo] add file   --增加文件
(
    NAME=F06_HotelPrice,
    FILENAME='d:\db\data\F06_HotelPrice.ndf',
    SIZE=1MB,
    FILEGROWTH=10%
)
TO filegroup FG06_HotelPrice;

alter database  [YeeSkyGo] add file   --增加文件
(
    NAME=F07_HotelPrice,
    FILENAME='d:\db\data\F07_HotelPrice.ndf',
    SIZE=1MB,
    FILEGROWTH=10%
)
TO filegroup FG07_HotelPrice;

alter database  [YeeSkyGo] add file   --增加文件
(
    NAME=F08_HotelPrice,
    FILENAME='d:\db\data\F08_HotelPrice.ndf',
    SIZE=1MB,
    FILEGROWTH=10%
)
TO filegroup FG08_HotelPrice;

alter database  [YeeSkyGo] add file   --增加文件
(
    NAME=F09_HotelPrice,
    FILENAME='d:\db\data\F09_HotelPrice.ndf',
    SIZE=1MB,
    FILEGROWTH=10%
)
TO filegroup FG09_HotelPrice;

alter database  [YeeSkyGo] add file   --增加文件
(
    NAME=F10_HotelPrice,
    FILENAME='d:\db\data\F10_HotelPrice.ndf',
    SIZE=1MB,
    FILEGROWTH=10%
)
TO filegroup FG10_HotelPrice;

alter database  [YeeSkyGo] add file   --增加文件
(
    NAME=F11_HotelPrice,
    FILENAME='d:\db\data\F11_HotelPrice.ndf',
    SIZE=1MB,
    FILEGROWTH=10%
)
TO filegroup FG11_HotelPrice;

go

--创建分区函数
CREATE PARTITION FUNCTION PF_YeeSkyGo_HotelPrice_DateRange(datetime)  AS RANGE RIGHT FOR VALUES (N'2018-07-01 00:00:00.000', 
																		      N'2018-09-01 00:00:00.000',
																			  N'2018-11-01 00:00:00.000',
																			  N'2019-01-01 00:00:00.000',
																			  N'2019-03-01 00:00:00.000',
																			  N'2019-05-01 00:00:00.000',
																			  N'2019-07-01 00:00:00.000',
																			  N'2019-09-01 00:00:00.000',
																			  N'2019-11-01 00:00:00.000',
																			  N'2020-01-01 00:00:00.000');


GO

--分区架构
CREATE PARTITION SCHEME [PS_YeeSkyGo_HotelPrice_DateRange] AS PARTITION [PF_YeeSkyGo_HotelPrice_DateRange] TO 
 (FG01_HotelPrice, FG02_HotelPrice, FG03_HotelPrice, FG04_HotelPrice, FG05_HotelPrice,
  FG06_HotelPrice, FG07_HotelPrice, FG08_HotelPrice, FG09_HotelPrice, FG10_HotelPrice, FG11_HotelPrice);
GO

/*****
* 这个表拥有一般普通表的特性——有主键，同时这个主键还是聚集索引。
* 分区表是以某个字段为分区条件，所以，除了这个字段以外的其他字段，是不能创建聚集索引的。
* 因此，要想将普通表转换成分区表，就必须要先删除聚集索引，然后再创建一个新的聚集索引，在该聚集索引中使用分区方案。
* 在SQL Server中，如果一个字段既是主键又是聚集索引时，并不能仅仅删除聚集索引。因此，我们只能将整个主键删除，然后重新创建一个主键，只是在创建主键时，不将其设为聚集索引.
*/

--删掉主键  
ALTER TABLE [T_GWZJ_Hotel_Price] DROP CONSTRAINT [PK_dbo.T_GWZJ_Hotel_Price];
go
--创建主键，但不设为聚集索引  
ALTER TABLE [T_GWZJ_Hotel_Price] ADD CONSTRAINT [NonClusteredIndex_T_GWZJ_Hotel_Price] PRIMARY KEY NONCLUSTERED  
(  
    [ID] ASC  
) ON [PRIMARY];
go

--创建一个新的聚集索引，在该聚集索引中使用分区方案  
CREATE CLUSTERED INDEX [ClusteredIndex_T_GWZJ_Hotel_Price] ON [T_GWZJ_Hotel_Price]([BizDate])  
ON [PS_YeeSkyGo_HotelPrice_DateRange]([BizDate]) ;
go
```
