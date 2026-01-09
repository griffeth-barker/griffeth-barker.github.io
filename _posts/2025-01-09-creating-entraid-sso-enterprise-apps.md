---
title: "Creating Multiple EntraID Enterprise Apps with SAML Single Sign-On"
tags:
  - powershell
  - entraid
  - microsoft-graph
  - saml
  - sso
categories:
  - powershell
---

## The Need
Over the last 6 or so years and across multiple organizations, I've had the need to create Enterprise Apps in Microsoft EntraID, create some security groups, assign them to the application, set an owner of each of the groups, etc. If you just do this once, it's not a huge deal to use the EntraID portal. But when you have a batch of them you need to do,
repeatedly, PowerShell and the Microsoft Graph API become things you really want to move to.

## The Solution
I decided to throw together this function to make my life a bit easier. This might be a bit more specific to my exact needs, but maybe someone else will find part of it helpful.
Essentially, the function takes in a name for the application, an identifier URI, a SAML response/ACS URL, the intended owner of the users group, and optionally a note about the groups. 
Be sure that you're connected to the Graph API using `Connect-MgGraph`, that you have either the (**Cloud Application Administrator** AND **Groups Administrator**) roles OR you are a **Global Administrator** (though, hopefully you're not doing too much day-to-day work on a GA account...). Also ensure you are connected with the following scopes:  
  - Group.ReadWrite.All  
  - Application.ReadWrite.All  
  - AppRoleAssignment.ReadWrite.All
  
The function creates two groups: one for administrators of the application and one for users of the application. The names of these groups are in a standard format that is `$Name Users` or `$Name Admins`. This helps with consistency when searching for these related components across EntraID. The intended owner of the users group as provided in the parameter is made the owner of that group, which is helpful for enabling managers or other teams to self-manage those groups instead of requiring intervention from the IT team.
An app registration is created along with an enterprise app and they are associated, then the groups are tied to the enterprise application. When these are created, they have sign-on preference set to SAML and the provided identifier and ACS URLs are configured.

The output of this function is just a PSCustomObject that displays, for each of the apps created:
```output
ApplicationId         : 00000000-0000-0000-0000-000000000000
ApplicationName       : grifftestapp1
SamlIdentifier        : https://grifftestapp1.domain.tld     
SamlReplyUri          : https://grifftestapp1.domain.tld/saml/acs
SamlMetadataUrl       : https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000/federationmetada
                        ta/2007-06/federationmetadata.xml?appid=00000000-0000-0000-0000-000000000000
UserGroupObjectId     : 00000000-0000-0000-0000-000000000000
UserGroupDisplayName  : grifftestapp1 Users
AdminGroupObjectId    : 00000000-0000-0000-0000-000000000000
AdminGroupDisplayName : grifftestapp1 Admins

ApplicationId         : 00000000-0000-0000-0000-000000000000
ApplicationName       : grifftestapp2
SamlIdentifier        : https://grifftestapp2.domain.tld     
SamlReplyUri          : https://grifftestapp2.domain.tld/saml/acs
SamlMetadataUrl       : https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000/federationmetada
                        ta/2007-06/federationmetadata.xml?appid=00000000-0000-0000-0000-000000000000
UserGroupObjectId     : 00000000-0000-0000-0000-000000000000
UserGroupDisplayName  : grifftestapp2 Users
AdminGroupObjectId    : 00000000-0000-0000-0000-000000000000
AdminGroupDisplayName : grifftestapp2 Admins
```

## Usage
Using the function is simple:
### Example 1
Create a single enterprise app with SAML SSO and associated groups:
```powershell
    $appParams = @{
        Name = 'App Name'
        IdentifierUri = 'https://example.domain.tld/app'
        ReplyUri = 'https://example.domain.tld/saml/acs'
        OwnerUPN = 'admin@domain.tld'
        Note = 'Requested by person in reference #12345'
    }
    New-GJBEntraAppDeployment @appParams
```

### Example 2
Create a bunch of enterprise apps with SAML SSO and associated groups based on data in a CSV file (just make your columns headers the parameter names):
```powershell
Import-Csv -Path "C:\path\to\appsToCreate.csv" | New-GJBEntraAppDeployment
```
## Wrapping Up
I made sure that this would support working in the pipeline specifically because I wanted to be able to easily read in a bunch of group data from a CSV file to create the groups (the result of which is shown above), and easily pipe it back out to `Export-CSV` when done to get the information needed to distribute to folks.  

🔗 **You can find the full function in this [GitHub gist](https://gist.github.com/griffeth-barker/c72ef9d62fae3f1016bee6d41ef6d7c5).**

Have you run into a situation where you needed to repeatedly and consistently create enterprise apps in EntraID? What other features would you add to this function without making it too big? If you found this function helpful, I'd appreciate you giving the gist a ⭐ star, and I'd love to hear from you on [BlueSky](https://bsky.app/profile/griff.systems)!

