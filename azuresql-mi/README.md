# Azure SQL Managed Instance

## COPY_ONLY Backup with Azure Storage Account

[Restore a database to Azure SQL Managed Instance with SSMS](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/restore-sample-database-quickstart?view=azuresql-mi)

> **CREDENTIAL** must match the container path, begin with https,
> and can't contain a trailing forward slash.
> **IDENTITY** must be **SHARED ACCESS SIGNATURE**.
> **SECRET** must be the shared access signature token and **can't contain a leading ?**.

[How to move Azure SQL managed instance cross subscriptions](https://techcommunity.microsoft.com/t5/azure-database-support-blog/how-to-move-azure-sql-managed-instance-cross-subscriptions/ba-p/3710336)

[Export to a BACPAC file - Azure SQL Database and Azure SQL Managed Instance](https://learn.microsoft.com/en-us/azure/azure-sql/database/database-export?view=azuresql)

> Limitations:
> Azure SQL Managed Instance doesn't currently support exporting a database to a BACPAC
> file using the Azure portal or Azure PowerShell.
> To export a managed instance into a BACPAC file,
> **use SQL Server Management Studio (SSMS) or SQLPackage**.

[Tutorial: Configure replication between two managed instances](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/replication-between-two-instances-configure-tutorial?view=azuresql-mi)

[Copy-only backups](https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/copy-only-backups-sql-server?view=azuresqldb-mi-current)

[Take a COPY_ONLY backup of TDE protected database on Azure SQL Managed Instance](https://techcommunity.microsoft.com/t5/azure-sql-blog/take-a-copy-only-backup-of-tde-protected-database-on-azure-sql/ba-p/643407)

From the above link:

```sql
-- Check if the db is encrypted with TDE:
SELECT * FROM sys.dm_database_encryption_keys

-- If the db is encrypted, alter the db to turn off encryption.
-- Make sure there is no active transaction when performing this operation:
ALTER DATABASE MyDatabase SET ENCRYPTION OFF

-- Drop the database encryption key (DEK):
DROP DATABASE ENCRYPTION KEY

-- Truncate the Log:
DBCC SHRINKFILE (<logName>, 1)

-- This should not show any active VLF that is encrypted by thumbprint.
SELECT * FROM sys.dm_db_log_info

-- Take a COPY_ONLY on Azure SQL Managed Instance	
-- Prerequisite to have write permissions
CREATE CREDENTIAL [https://mystorageaccount.blob.core.windows.net/myfirstcontainer]
WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
SECRET = 'sp=...' -- Enter your secret SAS token here. Omit the leading ?

BACKUP DATABASE MyDatabase
TO URL = 'https://mystorageaccount.blob.core.windows.net/myfirstcontainer/MyDatabaseBackup.bak'
WITH STATS = 5, COPY_ONLY;

-- Check the backup status
SELECT * FROM sys.dm_operation_status
WHERE resource_type = 0

-- Check the backup file in the storage account
RESTORE FILELISTONLY FROM URL = 'https://mystorageaccount.blob.core.windows.net/examples/MyDatabaseBackup.bak';

-- Restore to MyDatabase2
RESTORE DATABASE MyDatabase2 FROM URL = 'https://mystorageaccount.blob.core.windows.net/myfirstcontainer/MyDatabaseBackup.bak';

-- Check the status of the restore operation
SELECT session_id as SPID, command, a.text AS Query, start_time, percent_complete
   , dateadd(second,estimated_completion_time/1000, getdate()) as estimated_completion_time
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) a
WHERE r.command in ('BACKUP DATABASE','RESTORE DATABASE');
```
