---
title: "Copy Entra Group Members"
tags:
  - powershell
  - microsoft-entra
  - group-management
categories:
  - powershell
---

A real short share today. If you ever have the occasion where you want to take all of the members of a specific Entra group (or set of Entra groups), and add them as direct members of a different Entra group, you can do so with the `Copy-EntraGroupMember` function. I recently had a reason to do this and didn't find anything terribly useful returned from `Get-Command -Module Microsoft.Entra*` so I threw together this function. Here's a quick breakdown.

The function requires the **Microsoft.Entra** module.
```console
    #requires -Module Microsoft.Entra
```

You provide one or more GUIDs of Entra groups as the SourceGroupId parameter, and a single GUID as the DestinationGroupId parameter.
```powershell
    [CmdletBinding(SupportsShouldProcess = $true)]
    param (
        [Parameter(Mandatory = $true, ValueFromPipeline = $true, Position = 0)]
        [System.Guid[]]
        $SourceGroupId,

        [Parameter(Mandatory = $true)]
        [System.Guid]
        $DestinationGroupId
    )
```

Your connection to EntraId (via `Connect-Entra`) will need to have the "Group.ReadWrite.All" scope.
```powershell
    $apiConnection = Get-EntraContext
    $apiScopes = $apiConnection.Scopes
    if (-not $apiConnection) {
        throw "No connection to EntraID found. Please run Connect-Entra -Scopes Group.ReadWrite.All"
    }
    if ($apiScopes -notcontains "Group.ReadWrite.All") {
        throw "Connection to EntraID does not have the Group.ReadWrite.All scope. Please reconnect with the Group.ReadWrite.All scope."
    }
```

Essentially, for each of the source groups provided, their members will be iterated over and for each of them, an attempt will be made to try and add them as a direct member of the destination group via `Add-EntraGroupMember`.
```powershell
Add-EntraGroupMember -GroupId $DestinationGroupId -MemberId $sourceMember.Id -ErrorAction Stop
```

So after importing the function, all you need to do to copy members is:
```powershell
$copyParams = @{
    SourceGroupId      = '00000000-0000-0000-0000-000000000000'
    DestinationGroupId = '00000000-0000-0000-0000-000000000001'
    WhatIf             = $true
}
Copy-EntraGroupMember @copyParams
```
If you're wanting to preview the changes that would be made, you can leave `WhatIf = $true`.  
If you're wanting to actually make the changes, eliminate that line or change `WhatIf = $false`.

Here is the [GitHub gist](https://gist.github.com/griffeth-barker/8db06cef8fbc4882a798c8f9accaad66) containing the full function.

Just thought I'd share. If you find this helpful, I'd appreciate a star on the gist!
