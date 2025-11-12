---
title: "PSProfileWatcher v0.1.1 Release"
tags:
  - powershell
  - profile-management
  - utilities
  - security
  - hash
  - integrity-checking
categories:
  - powershell
---

![PSProfileWatcher-Icon-Original-NoBg-128x128.png](/post-images/PSProfileWatcher-Icon-Original-NoBg-128x128.png)

# Introduction
Have you ever opened your PowerShell profile to find things in in that you forgot about? Ever wonder if someone has slipped a little something into someone else's PowerShell profile to execute some code unbeknownst to the user? 

This thought process was the impetus for me throwing together PSProfileWatcher. This simple script module can be used to verify if any unexpected changes have been made to your PowerShell profile since your last known-good configuration.

# Overview
### Add-PSProfileCheck
Running this function adds several lines to the beginning of your PowerShell profile that will execute before anything else in the profile.
These lines import the PSProfileWatcher module and then call the `Test-PSProfileHash` function.

### Export-PSProfileHash
This function exports the hash of the current profile, as well as the contents of it. You should run this any time you make changes to your profile.

### Test-PSProfileHash
This function is called when you load your profile (i.e. when you launch PowerShell). It checks the expected profile hash based on the output of `Export-PSProfileHash` from earlier against the actual hash of the current profile.

![PSProfileWatcher-Screenshot-01.png](/post-images/PSProfileWatcher-Screenshot-01.png)

If the hashes match, the profile continues to load.
If the hashes do not match, you are provided a diff of the last known-good profile content output by `ExportPSProfileHash` and given the opportunity to either abort loading the profile, or continue to load it.

### Remove-PSProfileCheck
If you no longer wish to perform the profile check, you can remove the code from your profile by running `Remove-PSProfileCheck`. This is recommended if you are uninstalling the module.

# Getting the Module
You can find the module on the [PowerShell Gallery](https://www.powershellgallery.com/packages/PSProfileWatcher/0.1.1) and on [GitHub](https://github.com/griffeth-barker/PSProfileWatcher).

The latest release notes are available [here](https://github.com/griffeth-barker/PSProfileWatcher/releases/tag/v0.1.1).

# Wrapping Up
There very well may be something like this out there already. And there are certainly improvements to be made in this project. I just had the thought and liked the concept and thought I'd work it out. Future additions may include automatic cleanup of old files, tightening ACLs on the profile cache files, reverting to the last known-good profile configuration, etc. If this is at all useful to you, I'd appreciate any input! You can find me on BlueSky or open a pull request on the project.
