---
title: "Get a List of a Domain User's Recent Lockouts"
tags:
  - troubleshooting
  - powershell
  - active-directory
---

Unlocking a domain user's account is simple enough, but what about when that user is getting locked out repeatedly? What is causing it? This is a headache that has plagued many an IT professional.

This question is simple enough to answer if you turn to the terminal. You can run the below PowerShell script to obtain a list of recent lockouts for a domain user. 

```PowerShell
# Here we accept the username of the user to investigate as input
[string]$user = Read-Host "Username to investigate"

# Import the Active Directory module
Import-Module ActiveDirectory

# Get the domain controller that holds the PDC role
$PDC = (Get-ADDomainController -Discover -Service PrimaryDC).HostName

# Query the Security logs for 4740 events (account lockout)
$lockouts = Get-WinEvent -ComputerName "$PDC" -FilterHashtable @{LogName='Security'; Id=4740} |
Where-Object {$_.Properties[0].Value -eq $user} |
Select-Object TimeCreated,
    @{Name='Account Name' ; Expression={$_.Properties[0].Value}},
    @{Name='Workstation'; Expression={$_.Properties[1].Value}}

if ($null -eq $lockouts) {
    Write-Host "The user has no recent lockouts to display." -ForegroundColor Green
}
else {
    Write-Host "The user has the following recent lockouts:" -ForegroundColor Red
    $lockouts
}
```

The output looks something like this:
```output
TimeCreated             Account Name      Workstations
----------             ------------      ------------
10/14/2023 09:20:17 AM Username          WorkstationName
10/14/2023 08:13:05 AM Username          WorkstationName
```

This can save a significant amount of time instead of retracing steps and pouring over event logs manually. Why do that when you can use the terminal and have it do it for you?

## Get the Script
You can copy/paste the above script into a file and save it as a .ps1, or if you'd like you can download the script from my public GitHub repository to your user profile's Downloads folder using this command at a PowerShell terminal:

```PowerShell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/griffeth-barker/public/main/powershell/get_domain_user_lockouts.ps1" -OutFile "$($env:USERPROFILE)\Downloads\get_domain_user_loc
kouts.ps1"
```

As always, you should fully understand any command or script before running it in your environment. 

## Additional Resources
You can find additional information about domain user lockouts at the links below:
- [Windows Security Event 4740: A user account was locked out](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4740)
- [ActiveDirectory PowerShell Module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps)
