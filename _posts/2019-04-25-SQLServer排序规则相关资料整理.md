---
layout:     post
title:      SQL Server 排序规则相关资料整理
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

# 1. SQL Server 排序规则查看

```sql
--查看服务器排序规则(安装时指定的排序规则)
SELECT SERVERPROPERTY('COLLATION') AS ServerCollation,
       DATABASEPROPERTYEX('tempdb', 'COLLATION') AS TempdbCollation,
       DATABASEPROPERTYEX(DB_NAME(), 'COLLATION') AS CurrentDBCollation;

--查看数据库排序规则
USE master;
GO
SELECT name, collation_name
FROM   sys.databases;

--当前数据库是否大小写敏感
SELECT CASE WHEN N'A'=N'a' THEN N'不敏感' ELSE N'敏感' END
```

# 2. 修改服务器的排序规则

## 2.1 获取服务器排序规则

```sql
SELECT SERVERPROPERTY('COLLATION');
```

## 2.2 查看当前服务器数据库安装版本

```sql
SELECT @@VERSION
```

### 2.3 确认当前数据库默认安装版本及默认安装文件目录,请根据自己版本记录数据库默认安装目录。

- (SQLServer Service) 默认安装目录：
  - SQL Server 2008 C:\Program Files\Microsoft SQL Server\100\Setup Bootstrap\Release
  - SQL Server 2012 C:\Program Files\Microsoft SQL Server\110\Setup Bootstrap\SQLServer2012
  - SQL Server 2014 C:\Program Files\Microsoft SQL Server\120\Setup Bootstrap\SQLServer2014
  - SQL Server 2016 C:\Program Files\Microsoft SQL Server\140\Setup Bootstrap\SQLServer2016

### 2.4 使用管理员权限打开命令行窗口，并进入默认安装目录

### 2.5 停止SQLServer服务

```bat
net stop mssqlserver
```

### 2.6 运行语句修改数据库排序规则

```bat
Setup /QUIET /ACTION=REBUILDDATABASE /instancename=MSSQLSERVER /SQLSYSADMINACCOUNTS=操作系统管理员账号 /sapwd=密码 /sqlcollation=Latin1_General_CI_AS
```

- 格式说明:

```bat
Setup /QUIET
      /ACTION=REBUILDDATABASE
      /INSTANCENAME=InstanceName
      /SQLSYSADMINACCOUNTS=accounts
      / [ SAPWD= StrongPassword ]
      /SQLCOLLATION=CollationName
```

- 注意: accounts 用户使用电脑管理员用户，CollationName 为需要变更的排序格式

### 2.7 启动SQLServer服务

```bat
net start mssqlserver
```

## 3. 修改数据库排序规则

```sql
USE [master]
GO
ALTER DATABASE [TZToolDB] SET  SINGLE_USER WITH ROLLBACK IMMEDIATE
GO

ALTER DATABASE [TZToolDB] COLLATE Latin1_General_CI_AS
GO

USE [master]
GO

ALTER DATABASE [TZToolDB] SET  multi_user WITH ROLLBACK IMMEDIATE
GO
```

## 4. 修改表排序规则

```sql
USE [master]
GO
ALTER DATABASE [TZToolDB] SET  SINGLE_USER WITH ROLLBACK IMMEDIATE
GO

DECLARE @table NVARCHAR(128); --循环Item表名
DECLARE @column NVARCHAR(128); --循环Item字段名
DECLARE @type NVARCHAR(128); --对应字段的类型，char、nchar、varchar、nvarchar等
DECLARE @typeLenght NVARCHAR(128); --对应类型的长度，nchar、nvarchar需要将数值除于2
DECLARE @sql NVARCHAR(MAX); --要拼接执行的sql语句


SET ROWCOUNT 0;

SELECT NULL mykey,
       c.name,
       t.name AS [Table],
       c.name AS [Column],
       c.collation_name AS [Collation],
       TYPE_NAME(c.system_type_id) AS [TypeName],
       c.max_length AS [TypeLength]
INTO #temp
FROM sys.columns c
    RIGHT JOIN sys.tables t
        ON c.object_id = t.object_id
WHERE c.collation_name IS NOT NULL;
--      AND t.name = 'SysLog'
--      AND TYPE_NAME(c.system_type_id) = 'nvarchar';

SET ROWCOUNT 1;
UPDATE #temp
SET mykey = 1;

WHILE @@ROWCOUNT > 0
BEGIN
    SET ROWCOUNT 0;

    --每次查询第一条记录并赋值到对应变量中
    SELECT @table = [Table],
           @column = [Column],
           @type = TypeName,
           @typeLenght = TypeLength
    FROM #temp
    WHERE mykey = 1;

    --nchar、nvarchar需要将数值除于2
    IF CONVERT(INT, @typeLenght) > 0
       AND
       (
           @type = 'nvarchar'
           OR @type = 'nchar'
       )
    BEGIN
        SET @typeLenght = CONVERT(NVARCHAR(128), CONVERT(INT, @typeLenght) / 2);
    END;

    IF @typeLenght = '-1'
    BEGIN
        SET @typeLenght = N'max';
    END;


    --拼接sql，注意表名、字段名要带[]，避免Group等关键字
    SET @sql
        = N' ALTER TABLE [' + @table + N'] ALTER COLUMN [' + @column + N'] ' + @type + N'(' + @typeLenght
          + N') COLLATE Latin1_General_CI_AS';

    --Try执行
    BEGIN TRY
        EXEC (@sql);
    END TRY
    --Catch查询异常结果
    BEGIN CATCH
        SELECT @sql AS [ASL],
               ERROR_MESSAGE() AS msg;
    END CATCH;

    DELETE #temp
    WHERE mykey = 1;

    SET ROWCOUNT 1;

    UPDATE #temp
    SET mykey = 1;
END;

SET ROWCOUNT 0;

DROP TABLE #temp;


USE [master]
GO

ALTER DATABASE [TZToolDB] SET  multi_user WITH ROLLBACK IMMEDIATE
GO
```