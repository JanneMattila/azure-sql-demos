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

[Bring Your Own Key to Azure SQL Database Managed Instance TDE](https://bradleyschacht.com/bring-your-own-key-to-azure-sql-database-managed-instance-tde/)

[Download SQL Server Management Studio (SSMS)](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16)

[Move Azure SQL Managed Instance across subnets](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/vnet-subnet-move-instance?view=azuresql&tabs=azure-portal)

[Move Azure SQL Managed Instance across the virtual networks](https://techcommunity.microsoft.com/t5/azure-sql-blog/move-azure-sql-managed-instance-across-the-virtual-networks/ba-p/3672378)

[Cross-subscription support for SQL MI database copy and move - GA refresh!](https://techcommunity.microsoft.com/t5/azure-sql-blog/cross-subscription-support-for-sql-mi-database-copy-and-move-ga/ba-p/4016701)

> Jan 22 2024: "_We don't have plans for supporting cross-tenant operations on our short-term roadmap._"

[Copy-only backups](https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/copy-only-backups-sql-server?view=azuresqldb-mi-current)

[Take a COPY_ONLY backup of TDE protected database on Azure SQL Managed Instance](https://techcommunity.microsoft.com/t5/azure-sql-blog/take-a-copy-only-backup-of-tde-protected-database-on-azure-sql/ba-p/643407)

From the above links:

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

-- From:
-- https://bradleyschacht.com/bring-your-own-key-to-azure-sql-database-managed-instance-tde/
SELECT
	DB_NAME(database_id) AS database_name,
	encryption_state,
	CASE encryption_state
		WHEN 0 THEN 'No Encryption Key Present. Database Not Encrypted.'
		WHEN 1 THEN 'Database Unencrypted'
		WHEN 2 THEN 'Database Encryption in Progress'
		WHEN 3 THEN 'Database Encrypted'
		WHEN 4 THEN 'Encryption Key Change in Progress'
		WHEN 5 THEN 'Database Decryption in Progress'
		WHEN 6 THEN 'Certificate or Key Change in Progress'
		ELSE 'Unknown Status'
		END AS encryption_state_descriptoin,
	encryptor_thumbprint
FROM sys.dm_database_encryption_keys

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
