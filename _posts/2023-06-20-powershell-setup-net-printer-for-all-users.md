---
title: "Use PowerShell to setup a network printer for all users"
tags:
  - powershell
  - windows
  - printers
categories:
  - powershell
---

# Purpose
This brief article details the commands necessary to use PowerShell to add/install/map a network printer for all users of a computer.

# Prerequisites
The below actions require the executing user to have administrative rights to the workstation in question.

# Solution
## Establish a Remote PowerShell session

In an elevated PowerShell session (running as administrator), run the following command;
```powershell
Enter-PsSession -ComputerName HOSTNAME.FQDN
```
where `HOSTNAME.FQDN` is the fully-qualified domain name of the workstation in question.

_Example:_ desktop01.yourdomain.com

You _can_ actually execute this process using the short name of the computer instead of the FQDN.

## Add the network printer for all users
Once you’ve entered the remote PowerShell session successfully, run the following commands in sequence;

```powershell
RUNDLL32 PRINTUI.DLL,PrintUIEntry /ga /n\\PRINTSERVER\Shared-printer
```

where `PRINTSERVER` is the NETBIOS name of your print server from which the printer is shared and `Shared-printer` is the shared name of the printer you’d like to add/install/map.

When this is complete, the network printer should be added and available for all users on that workstation. This can be verified by having users logon and check their Devices and Printers window.

---
# Additional Reading
- [Microsoft: rundll32 printui.dll,PrintUIEntry](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/rundll32-printui)
