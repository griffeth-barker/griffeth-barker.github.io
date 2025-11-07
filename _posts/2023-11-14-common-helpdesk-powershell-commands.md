---
title: Common Helpdesk PowerShell Commands
tags:
  - powershell
  - helpdesk
---

Here is a small collection of PowerShell commands that might be helpful to any user administrators or helpdesk or service delivery technicians.

## Computers
These commands are related to interacting with workstations and servers.

### Get Serial Number of a Computer
```PowerShell
(Get-CimInstance -Class Win32_BIOS).SerialNumber
```

### Rename Computer
```PowerShell
Rename-Computer -NewName YourNewName -Restart -Force
```

### Join Computer to Domain
```PowerShell
Add-Computer -Credential (Get-Credential) -DomainName YourDomain.com -Restart -Force
```

### Restart Computer
```PowerShell
Restart-Computer -Force
```

### Empty Recycle Bin
```PowerShell
Clear-RecycleBin -DriveLetter C -Force
```

### Get Computer Information
```PowerShell
Get-ComputerInfo
```

You can also select specific computer information, such as uptime, by specifying that property in the `-Properties` parameter:

```PowerShell
Get-ComputerInfo -Property OsUptime
```

### Get Services
List all services:
```PowerShell
Get-Service | Format-Table -AutoSize
```

Get details of a specific service:
```PowerShell
Get-Service -Name NameOfService | Select-Object * | Format-Table -AutoSize
```

### Group Policy Update
Force a group policy client update (this must be run from somewhere where the Remote Server Administration Tools (RSAT) are installed):

```PowerShell
Invoke-GPUpdate -ComputerName workstation01 -Force
```

This is the same as using `gpupdate /force` in Command Prompt.

Get the resulting applied group policies of the workstation/user:

```PowerShell
Get-GPResultantSetOfPolicy -Computer workstation01 -User username -ReportType Html -Path "C:\temp\gpresult.html)"
```

This is the same as using `gpresult /h C:\temp\gpresult.html` in Command Prompt.

### Workstation Trust Relationships
To check if the workstation's trust with the domain has failed, use:

```Powershell
Test-ComputerSecureChannel
```

If everything is okay, this should return `True`. If it returns `False`, then the workstation has lost its trust relationship with the domain. You can attempt to resolve this by using:

```Powershell
Test-ComputerSecureChannel -Repair
```

I recommend resetting the computer account in Active Directory before performing this step.

### Run a Command On A Remote Computer
As you can imagine, it can be helpful to run a command on a remote computer, whether it is to restart that computer, get it's information, etc.
```PowerShell
Invoke-Command -ComputerName server01 -ScriptBlock {your_command_goes_here}
```

### Get Local Users
```PowerShell
Get-LocalUser
```

## Users
These commands are related to interacting with users.

### Unlock Domain User Account
```Powershell
Unlock-ADAccount -Identity UserName
```

### Reset Domain User Password
```PowerShell
Set-ADAccountPassword UserName -NewPassword (Read-Host "Enter the new password" -AsSecureString) –Reset
```

### Add Domain User to Domain Group
```PowerShell
Add-ADGroupMember -Identity GroupName -Members UserName
```

## Networking
These commands are related to networking on a computer.

### Get Network Adapters
```PowerShell
Get-NetAdapter | Format-Table -AutoSize
```

### Get IP Addresses
```PowerShell
Get-NetIPAddress | Format-Table -AutoSize
```

### Flush DNS Cache
```PowerShell
Clear-DnsClientCache
```
This is the same as doing `ipconfig /flushdns` in Command Prompt.

### Use a Specific Domain Name Suffix
```PowerShell
Set-DnsClient -InterfaceAlias YourInterfaceAlias -ConnectionSpecificSuffix YourDomain.com
```

### Set DNS Servers
```PowerShell
Set-DnsClient -InterfaceIndex -ServerAddresses ("dns1.yourdomain.com","dns2.yourdomain.com")
```

Check out [Better than Just Pinging](_site/posts/2023-10-21-better-than-just-pinging.md) as well.

## Printers
Check out [Administer Windows Core Print Servers](_site/posts/2023-11-14-administer-windows-core-print-servers.md), as the commands described there are relevant in general.

## Clipboard Management
For most commands, you can take the output and copy it to your clipboard using `Set-Clipboard`. As an example, let's get the computer information and copy it:

```Powershell
Get-ComputerInfo | Out-String | Set-Clipboard
```

Get what is stored in your clipboard:

```PowerShell
Get-Clipboard
```

## Conclusion
There are *so* many helpful commands, as PowerShell is a powerful method of Windows administration. I hope some of these are helpful to someone! As always, you should fully understand what a command or script does before running it in your environment. Taking the time to learn how to use the terminal to administer your users and workstations will, over time, save you quite a bit of time. 
