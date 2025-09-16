# Azure SQL

## Entra ID Authentication

[Microsoft Entra server principals](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-logins?view=azuresql)

[Tutorial: Create and utilize Microsoft Entra server logins](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-logins-tutorial?view=azuresql)

[Configure and manage Microsoft Entra authentication with Azure SQL](https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure?view=azuresql&tabs=azure-powershell)

[Database-level roles](https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles?view=azuresqldb-current)

[Connect to Azure SQL with Microsoft Entra authentication and SqlClient](https://learn.microsoft.com/en-us/sql/connect/ado-net/sql/azure-active-directory-authentication?view=sql-server-ver17)

```sql
-- Switch to the database
USE AdventureWorks;
GO

-- Create users for Entra ID groups
CREATE USER [AppDB - Read only] FROM EXTERNAL PROVIDER;
CREATE USER [AppDB - Ops] FROM EXTERNAL PROVIDER;

-- Create custom roles
CREATE ROLE db_readonly;
CREATE ROLE db_ops;

-- Add Entra ID group users to roles
ALTER ROLE db_readonly ADD MEMBER [AppDB - Read only];
ALTER ROLE db_ops ADD MEMBER [AppDB - Ops];

-- List all schemas
SELECT name AS SchemaName
FROM sys.schemas
ORDER BY name;

-- List all tables in SalesLT schema
SELECT TABLE_SCHEMA, TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'SalesLT'

-- Read-only group: SELECT on SalesLT schema
GRANT SELECT ON SCHEMA::SalesLT TO db_readonly;

-- Ops group: required schemas
GRANT SELECT, INSERT, UPDATE, DELETE ON SCHEMA::SalesLT TO db_ops;
GRANT SELECT, INSERT, UPDATE, DELETE ON SCHEMA::dbo TO db_ops;

-- Read-only group: SELECT on specific table
GRANT SELECT ON OBJECT::dbo.ErrorLog TO db_readonly;

-- Check role memberships
SELECT dp1.name AS RoleName, dp2.name AS MemberName
FROM sys.database_role_members drm
JOIN sys.database_principals dp1 ON drm.role_principal_id = dp1.principal_id
JOIN sys.database_principals dp2 ON drm.member_principal_id = dp2.principal_id;

-- Check role permissions
SELECT dp.name AS RoleName, 
       perm.permission_name, 
       perm.state_desc AS PermissionState,
       obj.name AS ObjectName,
       sch.name AS SchemaName
FROM sys.database_permissions perm
JOIN sys.objects obj ON perm.major_id = obj.object_id
JOIN sys.schemas sch ON obj.schema_id = sch.schema_id
JOIN sys.database_principals dp ON perm.grantee_principal_id = dp.principal_id
WHERE dp.type IN ('R', 'S');
```
