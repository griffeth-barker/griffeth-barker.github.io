---
title: "Uninstall Windows Copilot using PowerShell"
tags:
  - powershell
  - windows
  - copilot
  - ai
categories:
  - powershell
---

# Uninstall Windows Copilot using PowerShell
If you’ve recently updated Windows or purchased a new Windows computer, you might have noticed a new addition: Windows Copilot. While some folks find it helpful, others don’t want an AI assistant running in the background--especially with how aggressively and persistently Microsoft prompts you to interact with it. If you fall into the latter camp, this post is for you.

## What is Windows Copilot?
From Microsoft's own [website](https://www.microsoft.com/en-us/microsoft-copilot/for-individuals/do-more-with-ai/general-ai/what-is-copilot?form=MA13KP):
> Copilot is an AI-powered assistant that can help you browse the web and much more! This intelligent assistant is here to help inform, empower, and support you in both your personal and professional life. Whether you ask Copilot simple or complex questions; use it to jumpstart your research or creative projects; or request it to generate a summary of your work meetings, it’s here for you, wherever you go. Copilot takes AI assistance to a whole new level, with applications for any situation that you might need. With helpful features such as Copilot Daily, Voice, and Image Creator, Copilot is your go-to AI companion for anytime, anywhere.

## Why Remove Windows Copilot?
While the capabilities of Copilot are potentially useful, the way that Microsoft has gone about bothering and nagging users to interact with it can really get in the way of you using your computer. Not to mention if you accidentally hit the Copilot key on newer laptops, or Win+C instead of Ctrl+C on older ones, you'll get a nice big Copilot application window popping up over whatever you were doing. 

## The PowerShell Function
To make the process as painless as possible, I wrote a PowerShell function that uninstalls Copilot.

The script works by searching for any installed packages with “Copilot” in their name. It then attempts to remove those packages using the appropriate PowerShell commands. If you choose the “CurrentUser” scope, it will only uninstall Copilot for your account. If you go with “AllUsers,” it will try to remove it system-wide.

## How to Use the Script
➡️ The function is available in this [GitHub Gist](https://gist.github.com/griffeth-barker/8d9d883a2429da671d3f7b5094217d6b). ⬅️

Copy the function from the Gist and paste it into your elevated (run as Administrator) PowerShell terminal.

To uninstall Copilot for just your user account, run:
```PowerShell
Uninstall-WindowsCopilot -Scope "CurrentUser"
```

If you want to remove it for all users on the machine, use:

```PowerShell
Uninstall-WindowsCopilot -Scope "AllUsers"
```

If you don’t specify a scope, the function defaults to “CurrentUser.”

## What the Function Does (and Doesn’t Do)
The function uses `Get-AppxPackage` to search for any installed packages that have "Copilot" in their name. For each of the packages found, it attempts to remove them using `Remove-AppxPackage`. Whether this is done for just your user account or for all users on your machine depends on the value of the `-Scope` parameter you use, as I described earlier.

The function does not remove the "Copilot for Microsoft 365" package, as it is bundled with Microsoft 365 and managed separately.

## Wrapping it Up
Windows Copilot isn’t for everyone. Thankfully, PowerShell gives us the tools to take control of what’s installed on our systems. Whether you’re a power user or just someone who likes to keep tight control over what is installed on your machine, this function should help you easily get rid of Copilot.

This portable PowerShell function enables you to simply copy/paste to disable this new feature introduced by Microsoft. If you want Copilot back, you can re-enable it using `winget install 9NHT9RB2F4HD` if you have `winget` installed, or you can browse the Microsoft Store and install it again manually.

If you found this helpful, I'd appreciate it if you starred the GitHub Gist. Thank you!