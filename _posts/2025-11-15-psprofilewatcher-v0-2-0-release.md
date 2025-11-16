---
title: "PSProfileWatcher v0.2.0 Release"
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
Thinking about design is important, and as it turns out, I wasn't happy with where PSProfileWatcher v0.1.1 was at, despite it serving the minimum function I originally desired. The more I--along with GitHub Copilot--massaged code and added functionality, I realized what I basically wanted for this module to be was a snapshot management system for your PowerShell profile, with load-time hash checks.

So here we arrive at v0.2.0!

# Overview
The naming of the functions in the module has been overhauled to reflect the snapshot management approach.  
- `New-PSProfileSnapshot`  
- `Get-PSProfileSnapshot`  
- `Restore-PSProfileSnapshot`  
- `Remove-PSProfileSnapshot`  
- `Compare-PSProfileSnapshot`  
- `Register-PSProfileSnapshotTest`  
- `Unregister-PSProfileSnapshotTest`  
- `Test-PSProfileSnapshot`  
- `Clear-PSProfileSnapshotOrphans`  

With this comes new functions that extend functionality of the module, a metadata storage system using JSON files for snapshot documentation, GUID-based snapshot identification, pipeline interoperability, orphaned file cleanup, along with Pester tests. Because of the desire for pipeline interoperability, parameter names were standardized across functions as well, including the use of aliases where appropriate to maintain the original functionality. Some type issues were resolved as well.

As with any time you're using a large language model to assist or drive your development, sometimes goofy things sneak their way in. As it turns out, `Get-PSProfileSnapshot` was using `Format-Table` which was causing problems with the object returned by the function; this has been fixed.

# Wrapping Up
I don't know where exactly I'll go with this module. Maybe it's more of a bit of fun than it is something useful to anyone -- and that's okay! But I think it's a great opportunity to work with module design, thought processes, etc.

You can find the latest release in the usual places:
- [GitHub](https://github.com/griffeth-barker/PSProfileWatcher)
  - [Full release notes](https://github.com/griffeth-barker/PSProfileWatcher/releases/tag/v0.2.0)
- [PowerShell Gallery](https://www.powershellgallery.com/packages/PSProfileWatcher/0.2.0)

Feel free to give it a whirl! Your feedback is welcome. You can reach me on BlueSky ([@griff.systems](https://griff.systems/bluesky)) or can open an issue/PR on the repository. If you like the project, please consider giving it a ⭐ star on GitHub.

Many thanks!
