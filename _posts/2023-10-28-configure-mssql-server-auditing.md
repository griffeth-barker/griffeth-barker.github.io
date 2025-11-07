---
title: Configure Microsoft SQL Server Auditing
tags:
  - mssqlserver
  - auditing
  - powershell
  - configuration
  - security
---
If you are a system administrator who works with SQL Server, you know how important it is to audit your SQL Server instances and databases. Auditing allows you to track and record the activities that occur on your SQL Server environment, such as who accessed what data, when, and how. This can help you improve the security and compliance of your data, as well as troubleshoot and investigate any issues or incidents that may happen. 

However, you also know how challenging it can be to set up and manage SQL Server auditing, especially if you have to deal with multiple servers and databases. That’s why using PowerShell scripts can be a great solution for SQL Server auditing!

This blog post includes a script from my public GitHub repository that you can use to aid in the configuration of SQL server auditing in your environment.

# The Script
```PowerShell
<#
.SYNOPSIS
  This script creates MSSQL server audit objects in order to write audit logs to the Windows Security Log.
.DESCRIPTION
  This script utilizes the SqlServer PowerShell module to pass Transact-SQL statements to the target SQL server 
  instance, which creates the following SQL server objects:
    - Server Audit object on the master
    - Server Audit Specification object on the master
    - Database Audit Specification object on individual database(s)
.PARAMETER <Parameter_Name>
  None
.INPUTS
  None
.OUTPUTS
  None
.NOTES
  Updated by:     Barker, Griffeth (barkergriffeth@gmail.com)
  Change Date:    2023-03-28
  Purpose/Change: Initial development

  This script requires that the server have Microsoft SQL Server and the SqlServer PowerShell module installed.

  This script is intended to be run via Group Policy Object against Microsoft SQL servers. 
.EXAMPLE
  None
#>

# Timestamp function for logging
function Get-TimeStamp {
  return "{0:MM/dd/yy} {0:HH:mm:ss}" -f (Get-Date)
}

###############################################################################################################################
# SCRIPT SETUP                                                                                                                #
###############################################################################################################################

Write-Host "[$(Get-Timestamp)] Setting up..." 
Write-Host "[$(Get-Timestamp)] Checking for existence of C:\temp ..."
$WorkingDir = "C:\temp"
if (Test-Path -Path $WorkingDir){
Write-Host "[$(Get-Timestamp)] Validated working directory." 
}
else {
Write-Host "[$(Get-Timestamp)] Could not find C:\temp - creating it now..." 
New-Item -Path "C:\" -Name "temp" -ItemType "directory" -Force
Write-Host "[$(Get-Timestamp)] Created C:\temp - proceeding." 
}

$FileDate = Get-Date -Format 'yyyyMMdd-HHmm'
Start-Transcript -Path "C:\temp\configure_sql_auditing_$FileDate.log" -Force

###############################################################################################################################
# TRANSACT-SQL VIA SQLSERVER POWERSHELL MODULE                                                                                #
###############################################################################################################################

# Get a list of the databases on the server. This is used to create the Database Audit Specification on each database.
Write-Host "[$(Get-Timestamp)] Getting list of databases to configure..."
try {
  $DbsToAudit = @((Get-SqlDatabase -ServerInstance $($env:COMPUTERNAME) -ErrorAction Stop | Where-Object {$_.Name -ne 'tempdb'}).Name)
  Write-Host "[$(Get-Timestamp)] Got list of databases." 
}
catch {
  Write-Host "[$(Get-Timestamp)] Failed to get list of databases! This could be due to the Powershell module not loading properly,"
  Write-Host "[$(Get-Timestamp)] invalid permissions, or a variety of other reasons. The script will now exit."
  Stop-Transcript
  Exit 
}

# Create the Server Audit object using Transact-SQL passed via SqlServer PowerShell Module
Write-Host "[$(Get-Timestamp)] Creating the Server Audit Object [your_audit_name]..." 
try {
  $SvrAuditObj = @"
USE master ;
GO

CREATE SERVER AUDIT [your_audit_name]
TO APPLICATION_LOG ;
GO

ALTER SERVER AUDIT [your_audit_name]
WITH (STATE = ON) ;
"@
  Invoke-Sqlcmd -ServerInstance $($env:COMPUTERNAME) -Query $SvrAuditObj
  Write-Host "[$(Get-Timestamp)] Successfully created the Server Audit Object [your_audit_name]." 
}
catch {
  Write-Host "[$(Get-Timestamp)] Failed to create the Server Audit Object [your_audit_name]!" 
}

# Create the Server Audit Specification object using Transact-SQL passed via SqlServer PowerShell Module
Write-Host "[$(Get-Timestamp)] Creating the Server Audit Specification Object [your_audit_name_spec]..." 
try {
  $SvrAuditSpecObj = @"
CREATE SERVER AUDIT SPECIFICATION [your_audit_name_spec]
FOR SERVER AUDIT [your_audit_name]
	ADD ( FAILED_LOGIN_GROUP ),
	ADD ( APPLICATION_ROLE_CHANGE_PASSWORD_GROUP ),
	ADD ( AUDIT_CHANGE_GROUP ),
	ADD ( BACKUP_RESTORE_GROUP ),
	ADD ( DATABASE_CHANGE_GROUP ),
	ADD ( DATABASE_OBJECT_OWNERSHIP_CHANGE_GROUP ),
	ADD ( DATABASE_OBJECT_PERMISSION_CHANGE_GROUP ),
	ADD ( DATABASE_OPERATION_GROUP ),
	ADD ( DATABASE_OWNERSHIP_CHANGE_GROUP ),
	ADD ( DATABASE_PERMISSION_CHANGE_GROUP ),
	ADD ( DATABASE_PRINCIPAL_CHANGE_GROUP ),
	ADD ( DATABASE_PRINCIPAL_IMPERSONATION_GROUP ),
	ADD ( DATABASE_ROLE_MEMBER_CHANGE_GROUP ),
	ADD ( FAILED_DATABASE_AUTHENTICATION_GROUP ),
	ADD ( FAILED_LOGIN_GROUP ),
	ADD ( SCHEMA_OBJECT_CHANGE_GROUP ),
	ADD ( SCHEMA_OBJECT_OWNERSHIP_CHANGE_GROUP ),
	ADD ( SCHEMA_OBJECT_PERMISSION_CHANGE_GROUP ),
	ADD ( SERVER_OBJECT_CHANGE_GROUP ),
	ADD ( SERVER_OBJECT_OWNERSHIP_CHANGE_GROUP ),
	ADD ( SERVER_OBJECT_PERMISSION_CHANGE_GROUP ),
	ADD ( SERVER_OPERATION_GROUP ),
	ADD ( SERVER_ROLE_MEMBER_CHANGE_GROUP ),
	ADD ( SERVER_STATE_CHANGE_GROUP ),
	ADD ( STATEMENT_ROLLBACK_GROUP ) ;
GO

ALTER SERVER AUDIT SPECIFICATION [your_audit_name_spec]
WITH (STATE = ON) ;
"@
  Invoke-Sqlcmd -ServerInstance $($env:COMPUTERNAME) -Query $SvrAuditSpecObj -ErrorAction Stop
  Write-Host "[$(Get-Timestamp)] Successfully created the Server Audit Specification Object [your_audit_name_spec]."
}
catch {
  Write-Host "[$(Get-Timestamp)] Failed to create the Server Audit Specification Object [your_audit_name_spec]!"
}

# Create the Database Audit Specification objects on each database using Transact-SQL passed via SqlServer PowerShell Module
foreach ($DbToAudit in $DbsToAudit){
  Write-Host "[$(Get-Timestamp)] $DbToAudit - Creating the Database Audit Specification Object [your_audit_spec_name]..."
  try {
    $DbAuditSpecObj = @"
USE $DbToAudit ;
GO

CREATE DATABASE AUDIT SPECIFICATION [your_audit_spec_name]
FOR SERVER AUDIT [your_audit_name]
WITH (STATE = ON);
GO
"@
    Invoke-Sqlcmd -ServerInstance $($env:COMPUTERNAME) -Query $DbAuditSpecObj -ErrorAction Stop
    Write-Host "[$(Get-Timestamp)] $DbToAudit - Successfully created the Database Audit Specification Object [your_audit_spec_name]."
  }
  catch {
    Write-Host "[$(Get-Timestamp)] $DbToAudit - Failed to create the Database Audit Specification Object [your_audit_spec_name]!"
  }
}

###############################################################################################################################
# SCRIPT CLEANUP                                                                                                              #
###############################################################################################################################
Stop-Transcript
```

The `$SvrAuditObj` and `$SvrAuditSpecObj` can be customized to audit different actions on the database servers based on your environment and your needs. There is additional information on what you can audit in the **Additional Resources** section at the end of this post.
# Get the Script
You can copy/paste the above script into a file and save it as a .ps1, or if you'd like you can download the script from my public GitHub repository to your user profile's Downloads folder using this command at a PowerShell terminal:

```PowerShell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/griffeth-barker/public/main/powershell/configure_mssql_server_auditing.ps1" -OutFile "$($env:USERPROFILE)\Downloads\configure_mssql_server_auditing.ps1"
```

As always, you should fully understand any command or script before running it in your environment. 
# Deployment
There are various ways this script can be utilized. The script can be loaded into Systems Center Configuration Manager (SCCM) and run against individual servers or collections of servers from there. Alternatively, the script can be copied directly to a server and run locally. You can even place the script in your domain's NETLOGON share or another location accessible by your servers and then configure a Group Policy Object to have servers call the script each time they boot.

One other great thing about configuring SQL server auditing is that the events will be written to the Windows Event Log, which can then easily be forwarded into your SIEM of choice for monitoring and alerting.

Auditing your databases is an important part of your systems monitoring and security; PowerShell is a powerful scripting language that can help automate this configuration. I hope you enjoyed this blog post and gained some useful knowledge about configuring SQL server auditing using Windows PowerShell!

# Additional Resources
- [Create Server Audit & Server Audit Specification - SQL Server | Microsoft Learn](https://learn.microsoft.com/en-us/sql/relational-databases/security/auditing/create-a-server-audit-and-server-audit-specification?view=sql-server-ver16)
- [Create a server audit & database audit specification - SQL Server | Microsoft Learn](https://learn.microsoft.com/en-us/sql/relational-databases/security/auditing/create-a-server-audit-and-database-audit-specification?view=sql-server-ver16)
- [Get Started Querying with Transact-SQL - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/paths/get-started-querying-with-transact-sql/)