---
layout:     post
title:      SQL Server基础知识
subtitle:   培训用的
date:       2019-04-25
author:     Winds
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - SQL
    - Database
    - Base
---

>近期整理的SQL Server基础知识

# SQL Server 基础知识

目录：

<!-- TOC -->

- [一、前言](#一前言)
  - [1. SQL Server 历史和现状](#1-sql-server-历史和现状)
  - [2. 连接方式](#2-连接方式)
  - [3. 常见管理工具](#3-常见管理工具)
  - [4. 常见连接故障](#4-常见连接故障)
  - [5. SQL Server学习网站](#5-sql-server学习网站)
- [二、应用基础](#二应用基础)
  - [1. 创建数据库](#1-创建数据库)
  - [2. 创建表和索引](#2-创建表和索引)
  - [3. 修改表和索引](#3-修改表和索引)
  - [4. 添加记录](#4-添加记录)
  - [5. 修改记录](#5-修改记录)
  - [6. 删除记录](#6-删除记录)
  - [7. 注释和说明](#7-注释和说明)
  - [8. 存储过程和函数](#8-存储过程和函数)
  - [9. 事务处理](#9-事务处理)
  - [10. 游标](#10-游标)

<!-- /TOC -->

## 一、前言

### 1. SQL Server 历史和现状

|            版本             |   操作系统    |   年代    |
|:---------------------------:|:-------------:|:---------:|
|         1.0、1.1、4.2         |  Sybase OS/2  | 1989-1992 |
|       NT、6.0、7.0、2000       |    Windows    | 1992-2000 |
| 2005、2008、2008 R2、2014、2016 |    Windows    | 2005-2016 |
|            2017             | Windows、Linux |   2017    |

- SQL Server的版本号
  - 8.0 -> sql server 2000
  - 9.0 -> sql server 2005
  - 10.0 -> sql server 2008
  - 10.5 -> sql server 2008 R2
  - 11.0 -> sql server 2012
  - 12.0 -> sql server 2014

### 2. 连接方式

`ODBC、JDBC、BDE、OLE DB、ADO.NET`

### 3. 常见管理工具

- Microsoft SQL Server Management Studio (SSMS)
- PremiumSoft Navicat Premium
- JetBrains DataGrip
- DBeaver Community
- HeidiSQL
- Azure Data Studio
- Visual Studio Code + SQL Server Extensions

### 4. 常见连接故障

- 数据库服务未启动，打开服务面版启动服务
- 数据库网络服务未配置TCP/IP协议，在数据库配置工具打开TCP/IP协议后重启数据库服务
- 实例名不是默认的MSSQLSERVER，需要在连接地址后指定实例名（可在配置管理工具中查看实例名），如数据库实例名为SQL，连接时需要在服务器名或IP地址后指定实例名，例：192.168.1.11\SQL
- 未开启双重认证方式，在服务器端以本地管理员登陆数据库后修改认证方式未双重，即Windows认证 + SQL Server认证

### 5. SQL Server学习网站

- [微软SQL Server文档：https://docs.microsoft.com/zh-cn/sql](https://docs.microsoft.com/zh-cn/sql)
- [w3c的培训文档：https://www.w3cschool.cn/sqlserver](https://www.w3cschool.cn/sqlserver)

## 二、应用基础

### 1. 创建数据库

```sql
USE master;
GO

CREATE DATABASE Test
  ON
  (
    NAME = Test_dat,
    FILENAME = 'D:\Database\Testdat.mdf',
    SIZE = 10,
    MAXSIZE = UNLIMITED,
    FILEGROWTH = 5 )
  LOG ON
  (
    NAME = Test_log,
    FILENAME = 'D:\Database\Testlog.ldf',
    SIZE = 5MB,
    MAXSIZE = 1024MB,
    FILEGROWTH = 5MB )
  COLLATE Latin1_General_CI_AS
GO

ALTER DATABASE Test SET RECOVERY SIMPLE;
GO
```

### 2. 创建表和索引

```sql
USE Test;
GO

CREATE TABLE dbo.Dev
(
  DevId      INT           IDENTITY(1, 1) NOT NULL,
  DevName    NVARCHAR(200) NOT NULL,
  StockQty   DECIMAL(9, 2) NULL,
  Flag       BIT           NOT NULL DEFAULT( 1 ),
  DevType    SMALLINT      NOT NULL DEFAULT( 1 ),
  CreateUser NVARCHAR(10)  NOT NULL,
  CreateDate DATETIME      NOT NULL DEFAULT( GETDATE()),
  ModifyUser NVARCHAR(10)  NULL,
  ModifyDate DATETIME      NULL,
  CONSTRAINT PK_Dev_DevId PRIMARY KEY CLUSTERED( DevId ASC )
);
GO
```

```sql
USE Test;
GO

CREATE TABLE dbo.Dev
(
  DevId      INT           IDENTITY(1, 1) NOT NULL,
  DevName    NVARCHAR(200) NOT NULL,
  StockQty   DECIMAL(9, 2) NULL,
  Flag       BIT           NOT NULL DEFAULT( 1 ),
  DevType    SMALLINT      NOT NULL DEFAULT( 1 ),
  CreateUser NVARCHAR(10)  NOT NULL,
  CreateDate DATETIME      NOT NULL DEFAULT( GETDATE()),
  ModifyUser NVARCHAR(10)  NULL,
  ModifyDate DATETIME      NULL,
);
GO
--创建主键
ALTER TABLE dbo.Dev
ADD CONSTRAINT PK_Dev_DevID PRIMARY KEY CLUSTERED( DevId );
GO
--删除主键
ALTER TABLE dbo.Dev
DROP CONSTRAINT PK_Dev_DevID;
```

### 3. 修改表和索引

```sql
USE Test;
GO

--添加字段
ALTER TABLE dbo.Dev
ADD remark NVARCHAR(500) NULL;
--修改字段名
EXECUTE sp_rename 'dbo.Dev.remark','dbo.Dev.memo';
--修改字段类型
ALTER TABLE dbo.Dev
ALTER COLUMN memo NVARCHAR(300);
--添加有默认值的字段
ALTER TABLE dbo.Dev
ADD delflag INT NOT NULL DEFAULT 0;
--删除字段
ALTER TABLE dbo.Dev
DROP COLUMN delflag;

--删除约束
ALTER TABLE dbo.Dev
DROP CONSTRAINT [DF__Dev__delflag__276EDEB3]
--查询约束
EXECUTE sp_helpconstraint @objname='dbo.Dev'
--查询约束
SELECT * FROM sys.objects WHERE parent_object_id=OBJECT_ID('dbo.Dev') --所有
SELECT * FROM sys.default_constraints WHERE parent_object_id=OBJECT_ID('dbo.Dev') --默认
SELECT * FROM sys.check_constraints   WHERE parent_object_id=OBJECT_ID('dbo.Dev') --检查
SELECT * FROM sys.key_constraints     WHERE parent_object_id=OBJECT_ID('dbo.Dev') --键
--添加默认约束
ALTER TABLE dbo.Dev
ADD CONSTRAINT DF_Dev_CreateUser DEFAULT( 'Admin' )FOR CreateUser;
--添加检查约束
ALTER TABLE dbo.Dev
ADD CONSTRAINT CK_Dev_StockQtyCheck CHECK( StockQty BETWEEN 0 AND 10000 );
--添加唯一约束
ALTER TABLE dbo.Dev
ADD CONSTRAINT UQ_Dev_DevName UNIQUE(DevName);
```

```sql
USE Test;
GO
--查询索引
SELECT     i.object_id, i.name index_name, i.index_id, i.type_desc, i.is_unique, ic.column_id, c.name column_name,ic.is_descending_key
FROM       sys.indexes i
INNER JOIN sys.index_columns ic ON ic.object_id = i.object_id AND ic.index_id = i.index_id
INNER JOIN sys.columns c ON c.object_id = i.object_id AND c.column_id = ic.column_id
WHERE      i.object_id = OBJECT_ID('dbo.Dev');

--创建非聚集索引
CREATE NONCLUSTERED INDEX Index_Dev_DevName
ON dbo.Dev (DevName)
WITH(DROP_EXISTING=ON)
--删除索引
DROP INDEX Index_Dev_DevName ON dbo.Dev;
--创建唯一非聚集索引
CREATE UNIQUE INDEX Index_Dev_DevName
ON dbo.Dev (DevName)
WITH(DROP_EXISTING=ON)
```

### 4. 添加记录

```sql
--单条添加
INSERT INTO dbo.Dev( DevName, StockQty, Flag, DevType, CreateUser)
VALUES( N'烘干炉', 2, 1, 0, N'Admin');
```

| DevId | DevName | StockQty | Flag | DevType | CreateUser | CreateDate              | ModifyUser | ModifyDate | memo |
|-------|---------|----------|------|---------|------------|-------------------------|------------|------------|------|
| 1     | 烘干炉  | 2.00     | 1    | 0       | Admin      | 2019-04-23 14:22:21.237 | NULL       | NULL       | NULL |

```sql
--多条记录
INSERT INTO dbo.Dev( DevName, StockQty, Flag, DevType, CreateUser )
VALUES( N'烘干炉', 2, 1, 0, N'Admin' ),
      ( N'三坐标', 8, 1, 0, N'system' );
```

| DevId | DevName | StockQty | Flag | DevType | CreateUser | CreateDate              | ModifyUser | ModifyDate | memo |
|-------|---------|----------|------|---------|------------|-------------------------|------------|------------|------|
| 1     | 烘干炉  | 2.00     | 1    | 0       | Admin      | 2019-04-23 14:22:21.237 | NULL       | NULL       | NULL |
| 2     | 三坐标  | 8.00     | 1    | 0       | system     | 2019-04-23 14:22:21.237 | NULL       | NULL       | NULL |

```sql
--从其他表获取数据插入
INSERT INTO dbo.Dev( DevName, StockQty, Flag, DevType, CreateUser )
SELECT DevName, StockQty, Flag, DevType, CreateUser
FROM   dbo.Dev
WHERE  DevId > 0;
```

| DevId | DevName | StockQty | Flag | DevType | CreateUser | CreateDate              | ModifyUser | ModifyDate | memo |
|-------|---------|----------|------|---------|------------|-------------------------|------------|------------|------|
| 1     | 烘干炉  | 2.00     | 1    | 0       | Admin      | 2019-04-23 14:22:21.237 | NULL       | NULL       | NULL |
| 2     | 三坐标  | 8.00     | 1    | 0       | system     | 2019-04-23 14:22:21.237 | NULL       | NULL       | NULL |
| 3     | 烘干炉  | 2.00     | 1    | 0       | Admin      | 2019-04-23 14:38:43.140 | NULL       | NULL       | NULL |
| 4     | 三坐标  | 8.00     | 1    | 0       | system     | 2019-04-23 14:38:43.140 | NULL       | NULL       | NULL |

### 5. 修改记录

```sql
UPDATE dbo.Dev
SET    ModifyUser = N'张三', ModifyDate = GETDATE()
WHERE  DevId = 2;
```

| DevId | DevName | StockQty | Flag | DevType | CreateUser | CreateDate              | ModifyUser | ModifyDate              | memo |
|-------|---------|----------|------|---------|------------|-------------------------|------------|-------------------------|------|
| 1     | 烘干炉  | 2.00     | 1    | 0       | Admin      | 2019-04-23 14:22:21.237 | NULL       | NULL                    | NULL |
| 2     | 三坐标  | 8.00     | 1    | 0       | system     | 2019-04-23 14:22:21.237 | 张三       | 2019-04-23 14:43:20.760 | NULL |
| 3     | 烘干炉  | 2.00     | 1    | 0       | Admin      | 2019-04-23 14:38:43.140 | NULL       | NULL                    | NULL |
| 4     | 三坐标  | 8.00     | 1    | 0       | system     | 2019-04-23 14:38:43.140 | NULL       | NULL                    | NULL |

### 6. 删除记录

```sql
--有条件删除
DELETE dbo.Dev WHERE DevId=1
--清空表内所有数据，自增字段复位，并且不产生日志
TRUNCATE TABLE dbo.Dev
```

### 7. 注释和说明

```sql
--单行注释

/*
多行注释
*/

--添加表注释
EXECUTE sys.sp_addextendedproperty @name = N'MS_Description', @value = N'设备主数据', @level0type = N'USER',
                                   @level0name = N'dbo', @level1type = N'TABLE', @level1name = N'Dev',
                                   @level2type = NULL, @level2name = NULL;

--修改表注释
EXECUTE sys.sp_updateextendedproperty @name = N'MS_Description', @value = N'设备主数据', @level0type = N'USER',
                                      @level0name = N'dbo', @level1type = N'TABLE', @level1name = N'Dev',
                                      @level2type = NULL, @level2name = NULL;

--删除表注释
EXECUTE sys.sp_dropextendedproperty @name = N'MS_Description', @level0type = N'USER', @level0name = N'dbo',
                                    @level1type = N'TABLE', @level1name = N'Dev', @level2type = NULL,
                                    @level2name = NULL;

--添加字段注释
EXECUTE sys.sp_addextendedproperty @name = N'MS_Description', @value = N'设备ID', @level0type = N'SCHEMA',
                                   @level0name = N'dbo', @level1type = N'TABLE', @level1name = N'Dev',
                                   @level2type = N'COLUMN', @level2name = N'DevId';

--更新字段注释
EXECUTE sys.sp_updateextendedproperty @name = N'MS_Description', @value = N'设备ID', @level0type = N'SCHEMA',
                                      @level0name = N'dbo', @level1type = N'TABLE', @level1name = N'Dev',
                                      @level2type = N'COLUMN', @level2name = N'DevId';

--删除字段注释
EXECUTE sys.sp_dropextendedproperty @name = 'MS_Description', @level0type = N'SCHEMA', @level0name = N'dbo',
                                    @level1type = N'TABLE', @level1name = N'Dev', @level2type = N'COLUMN',
                                    @level2name = N'DevId';

```

### 8. 存储过程和函数

```sql
/******************************************************************************
  
  *Copyright(C)
  *FileName:     p_ProcedureName
  *Author:
  *Version:      1.00
  *Date:         2018-12-15
  *Description:  存储过程描述
  *Others:
  *Function List:
  *History:

*******************************************************************************/

CREATE PROCEDURE [dbo].[p_ProcedureName]
  @parameter1 INT,
  @parameter2 NVARCHAR(50)
AS
BEGIN
  SET NOCOUNT ON;

  --存储过程内容

END;
```

### 9. 事务处理

```sql
--声明错误相关变量
DECLARE @ErrorCode INT; --错误代码
DECLARE @ErrorSeverity INT; --错误级别
DECLARE @ErrorState INT; --错误状态
DECLARE @ErrorText NVARCHAR(500); --错误内容

BEGIN TRY
  --开始事务
  BEGIN TRAN;

  /*
      增、删、改代码块

      此处遇到错误会自动跳到CATCH
  */

  --逻辑处理语句，抛出自定义错误
  SET @ErrorCode = -3;
  SET @ErrorSeverity = 16;
  SET @ErrorState = 1;
  SET @ErrorText = N'错误内容';

  --抛出自定义错误，会被CATCH捕获
  RAISERROR(@ErrorText, @ErrorSeverity, @ErrorState);

  --提交事务
  COMMIT TRAN;

  --返回0
  RETURN 0;
END TRY
BEGIN CATCH
  --回滚事务
  ROLLBACK TRAN;

  SET @ErrorCode = ERROR_NUMBER();
  SET @ErrorSeverity = ERROR_SEVERITY();
  SET @ErrorState = ERROR_STATE();
  SET @ErrorText = ERROR_MESSAGE();

  --抛出错误
  RAISERROR(@ErrorText, @ErrorSeverity, @ErrorState);

  RETURN @ErrorCode;
END CATCH;
```

### 10. 游标

```sql
--声明变量
DECLARE @DevId INT; --设备ID
DECLARE @DevName NVARCHAR(200);

--设备名称
--声明游标
DECLARE c_Dev CURSOR FOR SELECT DevId, DevName FROM dbo.Dev;

--打开游标，获取首行数据给变量
OPEN c_Dev;

FETCH NEXT FROM c_Dev
INTO @DevId, @DevName;

--从全局变量获取游标状态，0表示已到表尾
WHILE @@FETCH_STATUS = 0
BEGIN
  --处理过程
  SELECT @DevId, @DevName;

  FETCH NEXT FROM c_Dev
  INTO @DevId, @DevName;
END;

--关闭游标
CLOSE c_Dev;
--释放游标资源
DEALLOCATE c_Dev;
```
