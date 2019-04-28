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

>SQL Server 运维知识库

# 日志清理

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