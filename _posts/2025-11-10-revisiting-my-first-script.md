---
title: "Revisiting My First Script"
tags:
  - powershell
  - workstation-configuration
  - windows
categories:
  - powershell
---

![](/post-images/revisiting-my-first-script-banner.png)

# Background
My initial foray into PowerShell came from having too much to do, and not enough time to do it all. In my first IT job (where I was the sole non-leadership IT person), I spent a lot of time setting up workstations. We didn't have SCCM, MDT, or any other common imaging and deployment tools, so I was manually doing this. Setting various Windows settings, installing software, running updates, etc. With various acquisitions that we had done, my workload was drastically increasing. I began my search for a way to make this process easier, without bringing in a new tool. Many years later now, I figured it could be good to revisit my first ever PowerShell script, as cringe as it may be. So let's brace ourselves and get to it! I'll truncate repetitive portions for brevity.

# The Script
First off, an observation. I used `Write-Host` *a lot*, including to print a welcome message to the console when running the script.
```powershell
Write-Host " "
Write-Host "#################################################################################"
Write-Host "# WELCOME TO THE WINDOWS 10 CONFIGURATION SCRIPT FOR COMPANY!                                            #"
Write-Host "# This script is intended to be run on all new computers and new computer images for COMPANY.            #"
Write-Host "# This script must be run in an elevated (administrative) PowerShell session.                            #"
Write-Host "# Pressing the Enter/Return key will continue with the script...                                         #"
Write-Host "##########################################################################################################"
Write-Host " "
Pause
```
This really isn't how I operate anymore, and I'd definitely do away with these 9 lines of code entirely.

Some of the setup for my script included setting the execution policy and creating a temp directory (again, printing messages to the console):
```powershell
# SET EXECUTION POLICY
Write-Host "Setting Execution Policy..."
Set-ExecutionPolicy Bypass -Force
Write-Host " "

# CREATE C:\ARCHIVE DIRECTORY
Write-Host "Creating 'C:\Archive' directory..."
New-Item -Path "C:\" -Name "Archive" -ItemType "directory"
Write-Host " "
```
Nowadays I'd probably just use either `$env:TEMP` or `[System.IO.Path]::GetTempPath()` instead of creating a whole new directory.

Next up it looks like I was disabling IPv6 and disabling Windows Firewall for the domain profile (I assure you, this was what I was instructed to do, not my preference):
```powershell
# DISABLE IPv6 FOR ALL INSTALLED NETWORK INTERFACES
Write-Host "Disabling IPv6 for all installed network interfaces..."
Get-NetAdapter | foreach { Disable-NetAdapterBinding -InterfaceAlias $_.Name -ComponentID ms_tcpip6 }
Write-Host " "

# DISABLE WINDOWS FIREWALL FOR DOMAIN NETWORKS
Write-Host "Disabling Windows Defender Firewall for the Domain network environment..."
Set-NetFirewallProfile -Profile Domain -Enabled False
```
I don't really have any comments for this (aside from a couple of global things, while I'll cover at the end of this section).

There was a long section of code dedicated to removing Windows 10 bloatware and undesired apps (highly truncated for brevity):
```powershell
$apps = @(
    # default Windows 10 apps
    "Microsoft.3DBuilder"
    "Microsoft.Appconnector"
    "Microsoft.BingFinance"
    "Microsoft.BingNews"
    "Microsoft.BingSports"
    # ... so very many more
)

foreach ($app in $apps) {
    Write-Output "Trying to remove $app"

    Get-AppxPackage -Name $app -AllUsers | Remove-AppxPackage -AllUsers

    Get-AppXProvisionedPackage -Online |
        Where-Object DisplayName -EQ $app |
        Remove-AppxProvisionedPackage -Online
}
```

 Oh, look, a wild `Write-Output` appeared amidst my `Write-Host`s! I guarantee I didn't understand the difference at the time. I was definitely referencing things I found online, since the next section of the script referenced a function that I did not include in the script, because I didn't really understand exactly what I was doing or how PowerShell worked at the time.
```powershell
# Multiple instances of something like this
# But I never defined the `force-mkdir` function anywhere...
force-mkdir "HKLM:\SOFTWARE\Policies\Microsoft\WindowsStore"
Set-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\WindowsStore" "AutoDownload" 2
# ...
```

Then there was this sick piece of code that supposedly downloaded the latest Adobe Reader DC installer. But it was just a `Write-Host` then a commented out Uri, so that's neat.

I then apparently tried to check for the **C:\archive** directory mentioned earlier, and tried to create it (again). For...reasons? To my own credit, I did actually manage to use an if/else statement at this point.

Oh, look, here's where that Adobe download was happening...
```powershell
# Download the installer
$source = "http://ardownload.adobe.com/pub/adobe/reader/win/AcrobatDC/1502320053/AcroRdrDC1502320053_en_US.exe"
$destination = "$workdir\adobeDC.exe"
Invoke-WebRequest $source -OutFile $destination

# Start the installation
Start-Process -FilePath "$workdir\adobeDC.exe" -ArgumentList "/sPB /rs"

# Wait XX Seconds for the installation to finish
Start-Sleep -s 35

# Remove the installer
rm -Force $workdir\adobe*
```
This is certainly not how I'd download software these days, but I also tend to avoid `Invoke-Webrequest -OutFile` unless its a very small file since historically this was a slow way to download files. Nevermind the fact that there is absolutely no way to know if it downloaded properly, installed properly, exited properly, etc.

I repeated a similar block for Google Chrome, then wrapped up with printing one last message to the console:
```powershell
# CONFIRM EXIT
Write-Host "The script has finished running. Pressing Enter/Return will close the script."
Pause
```

# Observations
Obviously there's a lot that I would do differently these days. First, I probably wouldn't be using a PowerShell script to do this in the first place, since I'd be using AutoPilot, Intune, MDT, SCCM, etc. But in the event that I did, such a script would certainly:
  - Include the standard comment-based help section at the top
  - Not include inline comments where it is obvious what is happening
  - Use the system's existing temp directory
  - Store the list of Appx packages to remove in a separate file in the same repository as the script (oh, and have the script in a repository to begin with for that matter)
  - Write to a log file instead of the console
  - Include `try/catch` blocks for error handling
  - Actually define functions that are called later in the script
  - Use Winget, chocolately, or another package manager instead of downloading installers from the web
  - Use proper indentation, formatting, etc.
  - Not use aliases and expand full property names instead of using positional parameters

# Wrapping Up
It's tought to look back and cringe at the first scripts we wrote, but I think it offers a moment to pause and reflect on the progress we've made. There's always something to be learned. When I wrote that first script, I never imagined that I would end up spending so much of my time writing PowerShell, that it would carry me through my career up to this point, or that I'd be writing this blog or participating in such a great community.

We all start somewhere! And wherever you're at, I hope seeing this *rough* first script of mine encourages you to continue your PowerShell journey. Want to talk first scripts? I'd love to hear from you [on BlueSky](https://griff.systems/bluesky).
