---
layout:     post
title:      SQL Server 用脚本导出表注释
subtitle:   日常维护
date:       2019-05-15
author:     Winds
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - SQL
    - Database
    - Base
---

>SQL Server 知识库

# 产生一个表的所有注释创建脚本

```sql

DECLARE @sqlstr NVARCHAR(MAX) = N'';
DECLARE @TableName NVARCHAR(255) = N'表名';
DECLARE @TableDesc NVARCHAR(255);
DECLARE @ColumnName NVARCHAR(255);
DECLARE @ColumnDesc NVARCHAR(255);
DECLARE @IsDrop BIT = 1; --是否产生删除注释的脚本

IF EXISTS ( SELECT *
            FROM   sys.extended_properties
            WHERE  major_id = OBJECT_ID(@TableName)
              AND  minor_id = 0 )
BEGIN
  SELECT @TableDesc = CONVERT(NVARCHAR(255), value)
  FROM   sys.extended_properties
  WHERE  major_id = OBJECT_ID(@TableName)
    AND  minor_id = 0;

  IF( @IsDrop = 1 )
  BEGIN
    SET @sqlstr = N'EXECUTE sys.sp_dropextendedproperty @name = N''MS_Description'', @level0type = N''USER'',';
    SET @sqlstr = @sqlstr + N'@level0name = N''dbo'',@level1type = N''TABLE'', @level1name = N''' + @TableName + N''',';
    SET @sqlstr = @sqlstr + N'@level2type = NULL, @level2name = NULL';
    --删除表注释
    PRINT @sqlstr;
  END;

  SET @sqlstr = N'EXECUTE sys.sp_addextendedproperty @name = N''MS_Description'', @value = N''' + @TableDesc
                + N''',@level0name = N''dbo'',';
  SET @sqlstr = @sqlstr + N'@level0type = N''USER'',@level1type = N''TABLE'', @level1name = N''' + @TableName + N''',';
  SET @sqlstr = @sqlstr + N'@level2type = NULL, @level2name = NULL';
  --创建表注释
  PRINT @sqlstr;
END;

DECLARE c_Column CURSOR FOR
  SELECT          CONVERT(NVARCHAR(255), a.name) ColumnName, CONVERT(NVARCHAR(255), b.value) ColumnDesc
  FROM            sys.columns a
  LEFT OUTER JOIN sys.extended_properties b ON b.major_id = a.object_id
                                           AND b.minor_id = a.column_id
  WHERE           a.object_id = OBJECT_ID(@TableName);

OPEN c_Column;

FETCH NEXT FROM c_Column
INTO @ColumnName, @ColumnDesc;

WHILE @@FETCH_STATUS = 0
BEGIN
  IF( @ColumnDesc <> N'' )
  BEGIN
    IF( @IsDrop = 1 )
    BEGIN
      SET @sqlstr = N'EXECUTE sys.sp_dropextendedproperty @name = ''MS_Description'', @level0type = N''SCHEMA'', @level0name = N''dbo'',';
      SET @sqlstr = @sqlstr + N'@level1type = N''TABLE'', @level1name = N''' + @TableName
                    + N''', @level2type = N''COLUMN'',';
      SET @sqlstr = @sqlstr + N'@level2name = N''' + @ColumnName + N'''';
      --删除列注释
      PRINT @sqlstr;
    END;

    SET @sqlstr = N'EXECUTE sys.sp_addextendedproperty @name = N''MS_Description'', @value = N''' + @ColumnDesc
                  + N''', @level0type = N''SCHEMA'',';
    SET @sqlstr = @sqlstr + N'@level0name = N''dbo'', @level1type = N''TABLE'', @level1name = N''' + @TableName
                  + N''',';
    SET @sqlstr = @sqlstr + N'@level2type = N''COLUMN'', @level2name = N''' + @ColumnName + N'''';
    --创建列注释
    PRINT @sqlstr;
  END;

  FETCH NEXT FROM c_Column
  INTO @ColumnName, @ColumnDesc;
END;

CLOSE c_Column;
DEALLOCATE c_Column;
```