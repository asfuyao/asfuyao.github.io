---
layout:     post
title:      SQL Server 系统维护脚本
subtitle:   日常维护
date:       2019-04-28
author:     Winds
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - SQL
    - Database
    - Base
---

>SQL Server 知识库

<!-- TOC -->
- [数据库维护](#数据库维护)
  - [查看版本号](#查看版本号)
  - [通过脚本附加数据库](#通过脚本附加数据库)
  - [日志清理-适用于2008以上版本](#日志清理-适用于2008以上版本)
  - [查看存储过程-视图-触发器-函数](#查看存储过程-视图-触发器-函数)
  - [修改用户自定义表类型](#修改用户自定义表类型)

# 数据库维护

## 查看版本号

```sql
SELECT @@VERSION
```

## 通过脚本附加数据库

```sql
EXEC sp_attach_db @dbname = N'DATA',
                  @filename1 = N'C:\XXDATA\DATA_data.mdf',
                  @filename2 = N'C:\XXDATA\DATA_log.ldf';
```

## 日志清理-适用于2008以上版本

```sql
USE [master]
GO
--设置数据库恢复模式设为SIMPLE
ALTER DATABASE DB_Name SET RECOVERY SIMPLE WITH NO_WAIT
GO
ALTER DATABASE DB_Name SET RECOVERY SIMPLE
GO
USE DB_Name
GO
--收缩日志，需要指定日志的逻辑名，可通过后面的语句查看逻辑名
DBCC SHRINKFILE (N'DB_Name_log' , 11, TRUNCATEONLY)
GO

USE [master]
GO
--设置数据库恢复模式为FULL
ALTER DATABASE  DB_Name SET RECOVERY FULL WITH NO_WAIT
GO
ALTER DATABASE DB_Name SET RECOVERY FULL
GO

--查看数据库文件的逻辑名
USE DB_Name
GO
SELECT file_id,name from sys.database_files;
```

## 查看存储过程-视图-触发器-函数

```sql
  SELECT a.name, b.[definition]
  FROM   sys.all_objects a, sys.sql_modules b
  WHERE  a.is_ms_shipped = 0
    AND  a.object_id = b.object_id
    AND  a.[type] = 'V'  --脚本类型，P存储过程、V视图、TR触发器、AF聚合函数
    AND  a.is_ms_shipped = 0  --是否为系统脚本，0为用户脚本
    AND  LEFT(name, 3) NOT IN ('SP_', 'XP_')  --可通过名称过滤，例如过滤SP和XP开头的存储过程
```

## 修改用户自定义表类型

```sql
USE [TZToolDB]
GO
--首先要重命名表类型
EXEC sys.sp_rename 'tt_ToolStockInDetail','tt_ToolStockInDetail_bak'

--建立新的表类型
CREATE TYPE [dbo].[tt_ToolStockInDetail] AS TABLE(
	[SortID] [BIGINT] NOT NULL,
	[ToolNumber] [dbo].[ToolNumber] NOT NULL,
	[SerialNo] [dbo].[SerialNo] NOT NULL,
	[WarehouseID] [INT] NOT NULL,
	[Amount] [INT] NOT NULL,
	[QCDate] NVARCHAR(20) NULL,
	[Remark] [dbo].[Remark] NULL,
	PRIMARY KEY CLUSTERED 
(
	[ToolNumber] ASC
)WITH (IGNORE_DUP_KEY = OFF)
)
GO

--查询所有引用此表类型的存储过程、视图、函数
SELECT     a.name, a.type, b.definition
FROM       sys.sysobjects a
INNER JOIN sys.sql_modules b ON a.id = b.object_id
WHERE      a.type IN ('P', 'V', 'FN')
  AND      b.definition LIKE ( '%tt_ToolStockInDetail%' );

--用修改方式打开这些存储过程、视图、函数，执行变更，即不需要改变内容，只是用alter方式执行修改即可
--相当于重新编译存储过程

--删除原表类型
DROP TYPE dbo.tt_ToolStockInDetail_bak
```