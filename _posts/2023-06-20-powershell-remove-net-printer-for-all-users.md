---
title: "Use PowerShell to remove a network printer for all users"
tags:
  - powershell
  - windows
  - printers
---

# Purpose
This brief article details the commands necessary to use PowerShell to remove a network printer for all users of a computer.

# Prerequisites
The below actions require the executing user to be an administrator of the remote workstation, or a Domain Administrator.

# Solution
## Establish a Remote PowerShell session
In an elevated PowerShell session (running as administrator), run the following command;
```PowerShell
Enter-PsSession -ComputerName HOSTNAME.FQDN
```
where `HOSTNAME.FQDN` is the fully-qualified domain name of the workstation in question.

_Example:_ `desktop01.yourdomain.com`  

You _can_ actually perform these commands without using the FQDN, and using the short name instead.

## Remove the network printer for all users
Once you’ve entered the remote PowerShell session successfully, run the following commands in sequence;
```PowerShell
Get-WmiObject -Class Win32_Printer | Where-Object {$_.Name -eq ‘\\PRINTSERVER\Shared-printer‘}
```
where `PRINTSERVER` is the NETBIOS name of your print server from which the printer is shared and `Shared-printer` is the shared name of the printer you’d like to remove. This command will return the printer that you’d like to remove, if it is installed for the user running the command. Even if the user running the command does not have this printer installed, this process will still work as intended.
```PowerShell
Get-WmiObject -Class Win32_Printer | Where-Object {$_.Name -eq ‘\\PRINTSERVER\Shared-printer‘}| foreach {$_.delete()}
```
again, where `PRINTSERVER` is the NETBIOS name of your print server from which the printer is shared and `Shared-printer` is the shared name of the printer you’d like to remove.

When this is complete, the network printer should be uninstalled/removed for all users on that workstation. This can be verified by having users logon and check their Devices and Printers window.