---
title: "hybrID Helps with Hybrid Hell"
tags:
  - powershell
  - windows-presentation-framework
  - xaml
  - active-directory
  - entraid
  - exchange-hybrid
  - identity-management
categories:
  - powershell
---

Many of us have worked in organizations that have been in a halfway state between their on-premesis Active Directory/Exchange Server infrastructure and Entra ID/Exchange Online. I'd wager there are probably more companies in this hybrid in-between situation than there are fully-cloud organizations at this point. Having worked for several of such companies myself, I've often had coworkers ask me, "where do I manage this user/group?"

There are two parts to this battle. The first is identifying where the object should be managed, and then you need to actually browse to the portals to manage the objects.

As one way of easing the pain of these issues, I thought to create [hybrID](https://github.com/griffeth-barker/hybrID) (recently released at v0.1.1 on GitHub). It’s a lightweight, locally-run PowerShell/WPF application designed to take in a UPN, sAMAccountName, or ObjectGUID and tell you where to manage the object as well as generate the exact, direct management URLs for that specific object.

Here is a quick overview of the utility.

![hybrID-screenshot-main-postsearch-tiny.png](/post-images/hybrID-screenshot-main-postsearch-tiny.png)

> Want more screenshots? You can find them [here](https://github.com/griffeth-barker/hybrID/blob/main/docs/screenshots.md).

## Features
  - Query an identity to see its status across Active Directory, Entra ID, and Exchange Server/Online.
  - Automatically determines if a mailbox is On-Premises, Exchange Online (EXO), or an EXO mailbox managed via On-Premises Remote Mailbox rules.
  - Generates dynamic, clickable URLs that drop the technician directly into the exact user or group profile blade in the Entra ID or Exchange Admin Centers.
  - Built on WPF featuring a dark theme, hover tooltips, and a dynamic status bar.
  - Frictionless Authentication: Silently caches Microsoft Graph API tokens to prevent repetitive login prompts.

I'll be blunt: this is not a feature rich project at this point. This is just an idea that has a prerelease. I'm hoping to work on it more in the coming months. With the advent of Microsoft now/soon supporting changing the source of authority for hybrid users and mailboxes, I may even add a feature to support changing the source of authority for the queried object. Who knows!

## Methodology
### Determining object state
hybrID queries your on-premises Active Directory using standard `Get-ADObject` cmdlets from the RSAT module. It checks attributes like targetAddress, msExchHomeServerName, and mail to evaluate where mail is managed. Simultaneously, it reaches out to Entra ID using the Microsoft Graph PowerShell SDK (specifically `Get-MgUser` and `Get-MgGroup`).

By analyzing the presence and attributes of the object across both environments, hybrID’s internal logic deduces the object's  state:
  - On-Premises Only: Found only in AD.
  - Cloud-Only: Found only in Microsoft Graph.
  - Hybrid/Synced: Found in both directories, meaning management must be handled carefully based on your organization's sync rules (e.g. managing attributes on-prem while the mailbox lives in Exchange Online).

This initial classification is critical because it dictates which management portals are relevant for the specific user or group you are looking at.

### Deep linking
Once hybrID knows where the object lives, it needs to tell you how to get there. hybrID uses template-driven approach for the URLs.

The application relies on a central configuration file (config.json). This JSON fileacts as a repository for base URL templates corresponding to the various management interfaces you use daily (Entra admin center, Exchange Online, On-Prem ECP, etc.).
These templates contain placeholders like `{id}` or `{fqdn}`.

In the Invoke-Search.ps1 function which handles finding and evaluating the object, the application takes the specific identifiers
it just retrieved during the search phase—like the Entra ObjectID or your on-premises domain—and dynamically injects them into the designated templates. The result is a fully formed, context-aware management URL tailored specifically to the object you searched for so you don't have to waste time clicking through dozens of pages to get to where you need to be.

### Project structure and user interface
PowerShell was, of course, my language of choice for this as it is with most of my projects. WPF and XAML are great partners with PowerShell, so I decided to go for a Model-View-Controller-ish structure. Each of the windows the app can create are defined as separate `.xaml` files, while private functions handle all of the app's functionality. The one public script `hybrID.ps1` invokes the application.

```text
hybrID/
├── assets/              # GitHub repository assets
├── build/               # Build scripts
├── docs/                # Project documentation
├── hybrID/              # Core Application Directory
│   ├── assets/          # Static app resources (icons, images)
│   ├── config/          # Application state (config.json)
│   ├── private/         # Internal PowerShell logic & helper functions
│   ├── public/          # Main controller script (hybrID.ps1)
│   └── ui/              # XAML presentation layer files
├── tests/               
├── .gitignore
└── README.md
```

## Wrapping Up
I'll be working on this a bit more in the future and already have some thoughts of things I'd like to fix or add to it. I'd love if you had a look at the project [on GitHub](https://github.com/griffeth-barker/hybrID) and if you find it interesting or helpful, give it a ⭐ star. Have you had woes managin hybrid Microsoft environments? I'd also like to hear from you [on BlueSky](https://bsky.app/profile/griff.systems)!
