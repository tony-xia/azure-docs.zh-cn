---
title: 时态表入门
description: 了解如何开始使用 Azure SQL 数据库和 Azure SQL 托管实例中的时态表。
services: sql-database
ms.service: sql-db-mi
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: how-to
author: bonova
ms.author: bonova
ms.reviewer: sstein
ms.date: 06/26/2019
ms.openlocfilehash: ea037d12417c8fad9d80b77df69285ed2c8df31b
ms.sourcegitcommit: 4bebbf664e69361f13cfe83020b2e87ed4dc8fa2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2020
ms.locfileid: "91618652"
---
# <a name="getting-started-with-temporal-tables-in-azure-sql-database-and-azure-sql-managed-instance"></a>开始使用 Azure SQL 数据库和 Azure SQL 托管实例中的时态表
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

时态表是 Azure SQL 数据库和 Azure SQL 托管实例中的可编程功能，可用于跟踪和分析数据更改的完整历史记录，而无需编写自定义代码。 时态表保存与时间上下文密切相关的数据，因此，只有特定时段内的存储事实才会解译为有效。 利用时态表的这种属性，可执行基于时间的有效分析，并从数据演变中获得见解。

## <a name="temporal-scenario"></a>时态表方案

本文演示了在应用程序方案中使用时态表的步骤。 假设要跟踪从头开始开发的新网站上的用户活动，或要通过用户活动分析扩展的现有网站上的用户活动。 在这个简化的示例中，我们假设一段时间内浏览过的网页数是需要在托管于 Azure SQL 数据库或 Azure SQL 托管实例上的网站数据库中捕获和监视的指标。 用户活动历史分析的目标是获取有关重新设计网站的意见，并为访客提供更好的体验。

此方案的数据库模型非常简单 - 用户活动指标以一个整数字段 **PageVisited** 表示，并与用户配置文件中的基本信息一起捕获。 此外，对于基于时间的分析，需要为每个用户保留一系列的行，其中每行代表特定时间段内特定用户访问过的网页数。

![架构](./media/temporal-tables/AzureTemporal1.png)

幸运的是，无需对应用进行任何操作即可维护此活动信息。 可以使用时态表将过程自动化：使你在网站设计过程中保有完全的弹性并节省更多的时间，从而将重心放在数据分析本身。 只需确保将 **WebSiteInfo** 表配置为[版本由系统控制的临时表](/sql/relational-databases/tables/temporal-tables#what-is-a-system-versioned-temporal-table)即可。 下面描述了在此方案中使用时态表的确切步骤。

## <a name="step-1-configure-tables-as-temporal"></a>步骤 1：将表配置为临时表

根据是要开始新的开发工作，还是升级现有的应用程序，可以创建临时表，或者通过添加临时属性来修改现有表。 一般情况下，用户方案可能混用了这两个选项。 使用 [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS)、[SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt) (SSDT)、[Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio) 或其他任何 Transact-SQL 开发工具执行以下操作。

> [!IMPORTANT]
> 建议始终使用最新版本的 Management Studio 以保持与 Azure SQL 数据库和 Azure SQL 托管实例的更新同步。 [更新 SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms)。

### <a name="create-new-table"></a>创建新表

在 SSMS 对象资源管理器中使用上下文菜单项“新建版本由系统控制的表”打开包含时态表模板脚本的查询编辑器，并使用“指定模板参数的值”(Ctrl+Shift+M) 来填充模板：

![SSMSNewTable](./media/temporal-tables/AzureTemporal2.png)

在 SSDT 中将新项添加到数据库项目时，请选择“时态表(版本由系统控制)”模板。 此时会打开表设计器，让你轻松指定表布局：

![SSDTNewTable](./media/temporal-tables/AzureTemporal3.png)

也可以通过直接指定 Transact-SQL 语句来创建时态表，如以下示例中所示。 请注意，每个临时表的必需元素为 PERIOD 定义以及引用将存储历史行版本的另一个用户表的 SYSTEM_VERSIONING 子句：

```sql
CREATE TABLE WebsiteUserInfo
(  
    [UserID] int NOT NULL PRIMARY KEY CLUSTERED
  , [UserName] nvarchar(100) NOT NULL
  , [PagesVisited] int NOT NULL
  , [ValidFrom] datetime2 (0) GENERATED ALWAYS AS ROW START
  , [ValidTo] datetime2 (0) GENERATED ALWAYS AS ROW END
  , PERIOD FOR SYSTEM_TIME (ValidFrom, ValidTo)
 )  
 WITH (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.WebsiteUserInfoHistory));
```

创建版本由系统控制的临时表时，会自动创建随附默认配置的历史记录表。 默认历史记录表包含期限列（结束、开始）上启用页压缩的聚集 B 树索引。 此配置非常适合使用临时表的大部分方案，特别是用于[数据审核](/sql/relational-databases/tables/temporal-table-usage-scenarios#enabling-system-versioning-on-a-new-table-for-data-audit)。

在此特定案例中，我们的目标是针对一段较长的数据历史记录以及较大的数据集，执行基于时间的趋势分析，因此历史记录表的存储选择为聚集列存储索引。 聚集列存储为分析查询提供极佳的压缩和性能。 时态表允许灵活且完全独立地在当前表和时态表中配置索引。

> [!NOTE]
> 业务关键、常规用途、高级层、标准层、S3 及更高版本中提供列存储索引。

以下脚本演示如何将历史记录表的默认索引更改为聚集列存储：

```sql
CREATE CLUSTERED COLUMNSTORE INDEX IX_WebsiteUserInfoHistory
ON dbo.WebsiteUserInfoHistory
WITH (DROP_EXISTING = ON);
```

时态表在对象资源管理器中以特定图标表示以便于识别，其历史记录表显示为子节点。

![AlterTable](./media/temporal-tables/AzureTemporal4.png)

### <a name="alter-existing-table-to-temporal"></a>将现有表更改为临时表

让我们探讨替代方案，其中 WebsiteUserInfo 表已存在，但不是针对保留更改历史记录而设计的。 在这种情况下，只需将现有表扩展为临时表即可，如以下示例中所示：

```sql
ALTER TABLE WebsiteUserInfo
ADD
    ValidFrom datetime2 (0) GENERATED ALWAYS AS ROW START HIDDEN  
        constraint DF_ValidFrom DEFAULT DATEADD(SECOND, -1, SYSUTCDATETIME())
    , ValidTo datetime2 (0)  GENERATED ALWAYS AS ROW END HIDDEN
        constraint DF_ValidTo DEFAULT '9999.12.31 23:59:59.99'
    , PERIOD FOR SYSTEM_TIME (ValidFrom, ValidTo);

ALTER TABLE WebsiteUserInfo  
SET (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.WebsiteUserInfoHistory));
GO

CREATE CLUSTERED COLUMNSTORE INDEX IX_WebsiteUserInfoHistory
ON dbo.WebsiteUserInfoHistory
WITH (DROP_EXISTING = ON);
```

## <a name="step-2-run-your-workload-regularly"></a>步骤 2：定期运行工作负荷

时态表的主要优点是，不需要以任何方式更改或调整网站就可以执行更改跟踪。 创建时态表后，每当对数据进行修改时，以前的行版本都会自动保存。

若要对此特定方案使用自动更改跟踪，只需在每次用户结束网站上的会话时更新列 **PagesVisited** ：

```sql
UPDATE WebsiteUserInfo  SET [PagesVisited] = 5
WHERE [UserID] = 1;
```

请务必注意，更新查询不需要知道实际操作进行的时间，也不需要知道如何保留历史数据以供将来分析使用。 Azure SQL 数据库和 Azure SQL 托管实例会自动处理这两个方面。 下图演示了如何在每次更新时生成历史记录数据。

![TemporalArchitecture](./media/temporal-tables/AzureTemporal5.png)

## <a name="step-3-perform-historical-data-analysis"></a>步骤 3：执行历史数据分析

现在，当启用版本由系统控制的临时表时，只需一个查询就能执行历史数据分析。 本文将提供一些解决常见分析方案的示例 - 若要了解所有详细信息，请浏览随 [FOR SYSTEM_TIME](/sql/relational-databases/tables/temporal-tables#how-do-i-query-temporal-data) 子句一起引入的各种选项。

若要查看按访问网页次数排序的前 10 个用户，请运行以下查询：

```sql
DECLARE @hourAgo datetime2 = DATEADD(HOUR, -1, SYSUTCDATETIME());
SELECT TOP 10 * FROM dbo.WebsiteUserInfo FOR SYSTEM_TIME AS OF @hourAgo
ORDER BY PagesVisited DESC
```

可轻松修改此查询，以分析一天前、一个月前或所需的任何过去时间点的站点访问记录。

若要执行前一天的基本统计分析，请使用以下示例：

```sql
DECLARE @twoDaysAgo datetime2 = DATEADD(DAY, -2, SYSUTCDATETIME());
DECLARE @aDayAgo datetime2 = DATEADD(DAY, -1, SYSUTCDATETIME());

SELECT UserID, SUM (PagesVisited) as TotalVisitedPages, AVG (PagesVisited) as AverageVisitedPages,
MAX (PagesVisited) AS MaxVisitedPages, MIN (PagesVisited) AS MinVisitedPages,
STDEV (PagesVisited) as StDevViistedPages
FROM dbo.WebsiteUserInfo
FOR SYSTEM_TIME BETWEEN @twoDaysAgo AND @aDayAgo
GROUP BY UserId
```

若要搜索特定用户在某个时间段的活动，请使用 CONTAINED IN 子句：

```sql
DECLARE @hourAgo datetime2 = DATEADD(HOUR, -1, SYSUTCDATETIME());
DECLARE @twoHoursAgo datetime2 = DATEADD(HOUR, -2, SYSUTCDATETIME());
SELECT * FROM dbo.WebsiteUserInfo
FOR SYSTEM_TIME CONTAINED IN (@twoHoursAgo, @hourAgo)
WHERE [UserID] = 1;
```

图形可视化对于临时查询特别方便，因为可以轻松、直观地显示趋势和使用模式：

![TemporalGraph](./media/temporal-tables/AzureTemporal6.png)

## <a name="evolving-table-schema"></a>不断演变的表架构

通常，开发应用时需要更改临时表架构。 为此，只需运行常规 ALTER TABLE 语句，Azure SQL 数据库或 Azure SQL 托管实例就会正确传播历史记录表的更改。 以下脚本演示如何添加要跟踪的其他属性：

```sql
/*Add new column for tracking source IP address*/
ALTER TABLE dbo.WebsiteUserInfo
ADD  [IPAddress] varchar(128) NOT NULL CONSTRAINT DF_Address DEFAULT 'N/A';
```

同样，可在工作负荷处于活动状态时更改列定义：

```sql
/*Increase the length of name column*/
ALTER TABLE dbo.WebsiteUserInfo
    ALTER COLUMN  UserName nvarchar(256) NOT NULL;
```

最后，可删除不再需要的列。

```sql
/*Drop unnecessary column */
ALTER TABLE dbo.WebsiteUserInfo
    DROP COLUMN TemporaryColumn;
```

或者，在已连接到数据库（联机模式）或正在开发数据库项目（脱机模式）时，使用最新的 [SSDT](/sql/ssdt/download-sql-server-data-tools-ssdt) 来更改临时表架构。

## <a name="controlling-retention-of-historical-data"></a>控制历史数据的保留期

使用版本由系统控制的临时表时，历史记录表可能比常规表更容易增大数据库大小。 大型以及不断增长的历史记录表可能会成为一个问题，这不单单体现在存储成本的增加上，而且还会降低临时查询的性能。 因此，针对管理历史记录表中的数据制定一个数据保留策略，是规划和管理每个临时表的生命周期的一个重要方面。 使用 Azure SQL 数据库和 Azure SQL 托管实例，可以通过以下方法来管理时态表中的历史数据：

- [表分区](/sql/relational-databases/tables/manage-retention-of-historical-data-in-system-versioned-temporal-tables#using-table-partitioning-approach)
- [自定义清理脚本](/sql/relational-databases/tables/manage-retention-of-historical-data-in-system-versioned-temporal-tables#using-custom-cleanup-script-approach)

## <a name="next-steps"></a>后续步骤

- 有关临时表的详细信息，请参阅签出 [临时表](/sql/relational-databases/tables/temporal-tables)。
- 访问第9频道收听 [客户时态实现成功案例](https://channel9.msdn.com/Blogs/jsturtevant/Azure-SQL-Temporal-Tables-with-RockStep-Solutions) ，观看 [实时时态演示](https://channel9.msdn.com/Shows/Data-Exposed/Temporal-in-SQL-Server-2016)。
