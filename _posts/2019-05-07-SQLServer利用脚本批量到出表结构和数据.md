---
layout:     post
title:      SQL Server 利用脚本批量到出表结构和数据
subtitle:   日常维护
date:       2019-05-07
author:     Winds
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - SQL
    - Database
    - Base
---

>SQL Server 知识库

# 利用脚本批量到出表结构和数据
* 注意：数据表的来源库需要批量替换代码中的“来源数据库名”

```sql
--设置目标库
USE Test;
GO

SET NOCOUNT ON;

DECLARE @table_name sysname; --表名，包含dbo
DECLARE @name sysname; --表名
DECLARE @keep_collation BIT = 0; --是否保持原表的排序规则，1是、0否
DECLARE @t_sqlstr NVARCHAR(MAX) = N''; --创建表的脚本
DECLARE @i_sqlstr NVARCHAR(MAX) = N''; --插入数据的脚本
DECLARE @i_column NVARCHAR(MAX) = N''; --列名集合，逗号分隔
DECLARE @object_name sysname;
DECLARE @object_id INT;

BEGIN TRY
  BEGIN TRAN;

  --创建游标，从源库获取表名
  DECLARE c_table_name CURSOR FOR
    SELECT   name
    FROM     来源数据库名.sys.objects
    WHERE    type = 'U'
      AND    name NOT IN ('sysdiagrams') --将要排除的表放入此处
    ORDER BY name;

  OPEN c_table_name;

  FETCH NEXT FROM c_table_name
  INTO @name;

  SELECT @table_name = 'dbo.' + @name; --要生成创建脚本的表名

  WHILE @@FETCH_STATUS = 0
  BEGIN
    SELECT @object_name = '[' + s.name + '].[' + o.name + ']', @object_id = o.[object_id]
    FROM   来源数据库名.sys.objects o WITH( NOWAIT )
    JOIN   来源数据库名.sys.schemas s WITH( NOWAIT )ON o.[schema_id] = s.[schema_id]
    WHERE  s.name + '.' + o.name = @table_name
      AND  o.[type] = 'U'
      AND  o.is_ms_shipped = 0;

    WITH
    index_column AS
    (
      SELECT ic.[object_id], ic.index_id, ic.is_descending_key, ic.is_included_column, c.name
      FROM   来源数据库名.sys.index_columns ic WITH( NOWAIT )
      JOIN   来源数据库名.sys.columns c WITH( NOWAIT )ON ic.[object_id] = c.[object_id]
                                                  AND ic.column_id = c.column_id
      WHERE  ic.[object_id] = @object_id
    ),
    fk_columns AS
    (
      SELECT k.constraint_object_id, cname = c.name, rcname = rc.name
      FROM   来源数据库名.sys.foreign_key_columns k WITH( NOWAIT )
      JOIN   来源数据库名.sys.columns rc WITH( NOWAIT )ON rc.[object_id] = k.referenced_object_id
                                                   AND rc.column_id = k.referenced_column_id
      JOIN   来源数据库名.sys.columns c WITH( NOWAIT )ON c.[object_id] = k.parent_object_id
                                                  AND c.column_id = k.parent_column_id
      WHERE  k.parent_object_id = @object_id
    )
    SELECT @t_sqlstr = N'CREATE TABLE ' + @object_name + CHAR(13) + N'(' + CHAR(13)
                       + STUFF(
      ( SELECT    CHAR(9) + ', [' + c.name + '] '
                  + CASE
                      WHEN c.is_computed = 1 THEN 'AS ' + cc.[definition]
                    ELSE
                      UPPER(tp.name)
                      + CASE
                          WHEN tp.name IN ('varchar', 'char', 'varbinary', 'binary') THEN
                            '(' + CASE
                                    WHEN c.max_length = -1 THEN 'MAX'
                                  ELSE CAST(c.max_length AS VARCHAR(5))
                                  END + ')'
                          WHEN tp.name IN ('nvarchar', 'nchar', 'ntext') THEN
                            '(' + CASE
                                    WHEN c.max_length = -1 THEN 'MAX'
                                  ELSE CAST(c.max_length / 2 AS VARCHAR(5))
                                  END + ')'
                          WHEN tp.name IN ('datetime2', 'time2', 'datetimeoffset') THEN
                            '(' + CAST(c.scale AS VARCHAR(5)) + ')'
                          WHEN tp.name = 'decimal' THEN
                            '(' + CAST(c.[precision] AS VARCHAR(5)) + ',' + CAST(c.scale AS VARCHAR(5)) + ')'
                        ELSE ''
                        END + CASE
                                WHEN c.collation_name IS NOT NULL
                                 AND @keep_collation = 1 THEN ' COLLATE ' + c.collation_name
                              ELSE ''
                              END + CASE
                                      WHEN c.is_nullable = 1 THEN ' NULL'
                                    ELSE ' NOT NULL'
                                    END + CASE
                                            WHEN dc.[definition] IS NOT NULL THEN ' DEFAULT' + dc.[definition]
                                          ELSE ''
                                          END
                      + CASE
                          WHEN ic.is_identity = 1 THEN
                            ' IDENTITY(' + CAST(ISNULL(ic.seed_value, '0') AS CHAR(1)) + ','
                            + CAST(ISNULL(ic.increment_value, '1') AS CHAR(1)) + ')'
                        ELSE ''
                        END
                    END + CHAR(13)
        FROM      来源数据库名.sys.columns c WITH( NOWAIT )
        JOIN      来源数据库名.sys.types tp WITH( NOWAIT )ON c.user_type_id = tp.user_type_id
        LEFT JOIN 来源数据库名.sys.computed_columns cc WITH( NOWAIT )ON c.[object_id] = cc.[object_id]
                                                                 AND c.column_id = cc.column_id
        LEFT JOIN 来源数据库名.sys.default_constraints dc WITH( NOWAIT )ON c.default_object_id != 0
                                                                    AND c.[object_id] = dc.parent_object_id
                                                                    AND c.column_id = dc.parent_column_id
        LEFT JOIN 来源数据库名.sys.identity_columns ic WITH( NOWAIT )ON c.is_identity = 1
                                                                 AND c.[object_id] = ic.[object_id]
                                                                 AND c.column_id = ic.column_id
        WHERE     c.[object_id] = @object_id
        ORDER BY  c.column_id
        FOR XML PATH(''), TYPE ).value('.', 'NVARCHAR(MAX)'), 1, 2, CHAR(9) + ' ')
                       + ISNULL(
                         ( SELECT CHAR(9) + ', CONSTRAINT [' + k.name + '] PRIMARY KEY ('
                                  + ( SELECT STUFF(
      ( SELECT ', [' + c.name + '] ' + CASE
                                         WHEN ic.is_descending_key = 1 THEN 'DESC'
                                       ELSE 'ASC'
                                       END
        FROM   来源数据库名.sys.index_columns ic WITH( NOWAIT )
        JOIN   来源数据库名.sys.columns c WITH( NOWAIT )ON c.[object_id] = ic.[object_id]
                                                    AND c.column_id = ic.column_id
        WHERE  ic.is_included_column = 0
          AND  ic.[object_id] = k.parent_object_id
          AND  ic.index_id = k.unique_index_id
        FOR XML PATH(N''), TYPE              ).value('.', 'NVARCHAR(MAX)'), 1, 2, '')) + ')' + CHAR(13)
                           FROM   来源数据库名.sys.key_constraints k WITH( NOWAIT )
                           WHERE  k.parent_object_id = @object_id
                             AND  k.[type] = 'PK' ), '') + N')' + CHAR(13)
                       + ISNULL(
                         ( SELECT ( SELECT CHAR(13) + 'ALTER TABLE ' + @object_name + ' WITH'
                                           + CASE
                                               WHEN fk.is_not_trusted = 1 THEN ' NOCHECK'
                                             ELSE ' CHECK'
                                             END + ' ADD CONSTRAINT [' + fk.name + '] FOREIGN KEY('
                                           + STUFF(( SELECT ', [' + k.cname + ']'
                                                     FROM   fk_columns k
                                                     WHERE  k.constraint_object_id = fk.[object_id]
                                                     FOR XML PATH(''), TYPE ).value('.', 'NVARCHAR(MAX)'), 1, 2, '')
                                           + ')' + ' REFERENCES [' + SCHEMA_NAME(ro.[schema_id]) + '].[' + ro.name
                                           + '] ('
                                           + STUFF(( SELECT ', [' + k.rcname + ']'
                                                     FROM   fk_columns k
                                                     WHERE  k.constraint_object_id = fk.[object_id]
                                                     FOR XML PATH(''), TYPE ).value('.', 'NVARCHAR(MAX)'), 1, 2, '')
                                           + ')'
                                           + CASE
                                               WHEN fk.delete_referential_action = 1 THEN ' ON DELETE CASCADE'
                                               WHEN fk.delete_referential_action = 2 THEN ' ON DELETE SET NULL'
                                               WHEN fk.delete_referential_action = 3 THEN ' ON DELETE SET DEFAULT'
                                             ELSE ''
                                             END
                                           + CASE
                                               WHEN fk.update_referential_action = 1 THEN ' ON UPDATE CASCADE'
                                               WHEN fk.update_referential_action = 2 THEN ' ON UPDATE SET NULL'
                                               WHEN fk.update_referential_action = 3 THEN ' ON UPDATE SET DEFAULT'
                                             ELSE ''
                                             END + CHAR(13) + 'ALTER TABLE ' + @object_name + ' CHECK CONSTRAINT ['
                                           + fk.name + ']' + CHAR(13)
                                    FROM   来源数据库名.sys.foreign_keys fk WITH( NOWAIT )
                                    JOIN   来源数据库名.sys.objects ro WITH( NOWAIT )ON ro.[object_id] = fk.referenced_object_id
                                    WHERE  fk.parent_object_id = @object_id
                                    FOR XML PATH(N''), TYPE ).value('.', 'NVARCHAR(MAX)')), '')
                       + ISNULL(
                           (( SELECT CHAR(13) + 'CREATE' + CASE
                                                             WHEN i.is_unique = 1 THEN ' UNIQUE'
                                                           ELSE ''
                                                           END + ' NONCLUSTERED INDEX [' + i.name + '] ON '
                                     + @object_name + ' ('
                                     + STUFF(
      ( SELECT ', [' + c.name + ']' + CASE
                                        WHEN c.is_descending_key = 1 THEN ' DESC'
                                      ELSE ' ASC'
                                      END
        FROM   index_column c
        WHERE  c.is_included_column = 0
          AND  c.index_id = i.index_id
        FOR XML PATH(''), TYPE         ).value('.', 'NVARCHAR(MAX)'), 1, 2, '') + ')'
                                     + ISNULL(
                                         CHAR(13) + 'INCLUDE ('
                                         + STUFF(( SELECT ', [' + c.name + ']'
                                                   FROM   index_column c
                                                   WHERE  c.is_included_column = 1
                                                     AND  c.index_id = i.index_id
                                                   FOR XML PATH(''), TYPE ).value('.', 'NVARCHAR(MAX)'), 1, 2, '')
                                         + ')', '') + CHAR(13)
                              FROM   来源数据库名.sys.indexes i WITH( NOWAIT )
                              WHERE  i.[object_id] = @object_id
                                AND  i.is_primary_key = 0
                                AND  i.[type] = 2
                              FOR XML PATH(''), TYPE ).value('.', 'NVARCHAR(MAX)')), '');

    EXEC sys.sp_executesql @t_sqlstr;

    PRINT CONVERT(NVARCHAR(30), GETDATE(), 21) + N': Create Table ' + @table_name;

    SET @i_sqlstr = N'';

    IF EXISTS ( SELECT *
                FROM   来源数据库名.sys.identity_columns
                WHERE  object_id = @object_id
                  AND  is_identity = 1 )
    BEGIN
      SET @i_sqlstr = @i_sqlstr + N' SET IDENTITY_INSERT ' + @table_name + N' ON ';
    END;

    SET @i_sqlstr = @i_sqlstr + N' INSERT INTO ' + @table_name + N' (';
    SET @i_column = ( SELECT '' + name + ', '
                      FROM   来源数据库名.sys.syscolumns
                      WHERE  id = @object_id
                      FOR XML PATH(''));
    SET @i_column = LEFT(@i_column, LEN(@i_column) - 1);
    SET @i_sqlstr = @i_sqlstr + @i_column + N')';
    SET @i_sqlstr = @i_sqlstr + N' select ' + @i_column + N' from 来源数据库名.' + @table_name;

    IF EXISTS ( SELECT *
                FROM   来源数据库名.sys.identity_columns
                WHERE  object_id = @object_id
                  AND  is_identity = 1 )
    BEGIN
      SET @i_sqlstr = @i_sqlstr + N' SET IDENTITY_INSERT ' + @table_name + N' OFF ';
    END;

    EXEC sys.sp_executesql @i_sqlstr;

    PRINT CONVERT(NVARCHAR(30), GETDATE(), 21) + N' Insert Data to ' + @table_name;

    FETCH NEXT FROM c_table_name
    INTO @name;

    SELECT @table_name = 'dbo.' + @name; --要生成创建脚本的表名
  END;

  CLOSE c_table_name;
  DEALLOCATE c_table_name;

  COMMIT TRAN;
END TRY
BEGIN CATCH
  ROLLBACK TRAN;

  PRINT ERROR_MESSAGE();
END CATCH;

```