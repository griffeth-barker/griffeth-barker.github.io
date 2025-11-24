---
title: "Finding Intersections of Entra Users' Group Memberships"
tags:
  - powershell
  - graph
  - entraid
  - groups
  - utilities
categories:
  - powershell
---

As someone who frequently performs administration activities in Microsoft Entra ID, I've frequently had the occasion where I've needed to provide access to an enterprise application and wanted to identify if there was an existing group that would be appropriate to use, or if I would need to create a new group. I've had other occasions as well where it would be helpful to know if several users shared memberhip of any groups. As an Anti-ClickOps evangelist, I absolutely despise pulling open multiple tabs of Entra ID and comparing group memberships of various users.

After some looking, I hadn't found a quick way to do this in the Microsoft.Entra or Microsoft.Graph modules, so I figured a custom function might be the way to go.

`Get-MgCommonGroups` consumes an array of Entra user identities in the form of either ObjectIDs or UserPrincipalNames (you can mix and match as well). 
```powershell
[CmdletBinding()]
param(
    [Parameter(Mandatory)]
    [string[]]$UserIdentities
)
```

It will check to see if you have the Microsoft.Graph module and throw an exception if you do not.

The function iterates over the provided user identities and gets the Ids of the groups of which they are members. Memberships for the users are mapped and compared.
```powershell
    $allSets = @()
    foreach ($id in $UserIdentities) {
        try {
            $ids = Get-UserGroupIds -Identity $id # This uses a helper function defined earlier in the main function, not shown here
            $allSets += ,$ids
        } catch {
            Write-Warning "Skipping $($id): $($_.Exception.Message)"
        }
    }

    if (-not $allSets) { return @() }

    $common = $null
    foreach ($set in $allSets) {
        if ($null -eq $common) {
            $common = [System.Collections.Generic.HashSet[string]]::new()
            foreach ($gid in $set) { $common.Add($gid) | Out-Null }
        } else {
            $temp = [System.Collections.Generic.HashSet[string]]::new()
            foreach ($gid in $set) { $temp.Add($gid) | Out-Null }
            $common.IntersectWith($temp)
        }
        if ($common.Count -eq 0) { break }
    }

    if ($common.Count -eq 0) { return @() }

```

The results are output as a PSCustomObject that includes the Id, Display Name, Description, and member count of each of which all specified users are a member.
Sample output:
```output
GroupId                              DisplayName                       Description         MemberCount
-------                              -----------                       -----------         -----------
00000000-0000-0000-0000-000000000000 All Users                         Everyone            2989
00000000-0000-0000-0000-000000000000 Another Group                     For a purpose       2167
00000000-0000-0000-0000-000000000000 CO Exec Leaders                   Execs Only          12
00000000-0000-0000-0000-000000000000 CO Seniors                        Company Seniors     117
00000000-0000-0000-0000-000000000000 Some Group                        A group!            1
```

Here is the [GitHub gist](https://gist.github.com/griffeth-barker/d423a8d4494df4013f52cc3e26347e3b) containing the full function.

Just thought I'd share. If you find this helpful, I'd appreciate a star on the gist!
