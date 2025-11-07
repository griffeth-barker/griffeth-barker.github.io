---
title: Customize Your PowerShell Profile
tags:
  - terminal
  - powershell
  - profiles
  - customization
  - dot-files
---
# Introduction
A PowerShell profile is a script that runs every time you start PowerShell and allows you to configure various aspects of your shell environment. You can use it to change the appearance of your console, pre-load modules and scripts, create aliases and functions, and more. By creating your own PowerShell profile, you can tailor PowerShell to your preferences and needs. In the following sections, I will explain how to create a PowerShell profile and how to customize it to your liking.

# What is a Profile?
A PowerShell profile is a script that runs when PowerShell starts. You can use the profile as a startup script to customize your environment. You can add commands, aliases, functions, variables, modules, PowerShell drives and more. You can also add other session-specific elements to your profile so they’re available in every session without having to import or re-create them. PowerShell supports several profiles for users and host programs. However, it doesn’t create the profiles for you. You need to create the profile files and directories manually or by using a command.

# Locating your Profile
Your profile is most likely stored in your user profile's Documents directory, in the WindowsPowerShell directory:

```PowerShell
"$($env:userprofile)\Documents\WindowsPowerShell\"
```

You can check to see if you already have profile script using this command:

```PowerShell
Test-Path -Path $Profile
```

If for some reason this returns `False`, you can create your profile script with this command:

```PowerShell
New-Item -Path $Profile -Type File -Force
```

There are different profiles for different console hosts, so if you do a lot of work in another application such as VS Code, PowerShell ISE, etc. then you may have or need multiple profiles which contain configuration specific to those hosts. The good news is that they are typically stored in the same location as your standard profile, and you can edit them similarly.

# My Profile
Here is a screenshot of Windows Terminal using one of my profiles at first load:

![[Pasted image 20231119105720.png]]

And here is a screenshot of the in-terminal editor that I use for when I just need to do some quick text editing or modify a config file and don't need VS Code for scripting:

![[Pasted image 20231119110042.png]]

Looks pretty slick! So let's take a look at how you can create yourself a comfortable and customized PowerShell environment.

# Customizing your Profile
Ultimately your profile can be edited via any means you'd use to edit a text file. You can browse to the location printed to the terminal when you use `$profile` and open that file in your favorite editor. This might be something as simple and native as Notepad, something easy and third-party like the revered Notepad++, or a flow-blown ISE such as VS Code. If you have Vim or Nano installed on your system, you can also use that to edit your profile directly in your terminal with `vim $profile` or `nano $profile` (this is my preferred method, but use whichever way works best for you)!
## Customize your Color Scheme
Your terminal emulator of choice may already have inbuilt options for changing the color scheme. Windows Terminal, ConEmu, and Termius all have this functionality. The inbuilt Windows PowerShell console host allows you to manually change its colors, but does not offer inbuilt color scheme presets.

Some time ago, Microsoft released the [ColorTool](https://devblogs.microsoft.com/commandline/introducing-the-windows-console-colortool/) on their Windows Terminal GitHub repository. This provides a simple way of setting the color scheme of your terminal if you don't have access to a terminal emulator that offers easy color scheme presets.

Download the latest release from their GitHub repository, extract it, and place the `ColorTool` directory somewhere safe that won't get deleted, you have permissions to, etc. Inside the directory is the executable as well as a directory with some color schemes in it. ColorTool supports `.itermcolors` files so you can add your own if you don't like one of the default ones.

To set your color scheme once, you can use this command:
```PowerShell
Start-Process -Path "C:\Path\to\ColorTool.exe" -Arguments "OneHalfDark.itermcolors"
```

You could take it a step further and add this to your profile script if you wanted to make sure your color scheme is always set as you desire.
## Customize your Font
Most terminal emulators will offer some kind of fonts options, most often supporting most of the default fonts installed on your system, so pick one you like if you don't like the default Consolas or other font. I highly recommend a monospaced font for your terminal.

If you want to take it a step further, you can install a special Nerd Font which adds support for a variety of other characters in the terminal. Check out [Nerd Fonts](https://www.nerdfonts.com/) and see what's available that suits your needs and wishes!

I personally am just using a font called [Mononoki NF](https://madmalik.github.io/mononoki/).
## Customize your Shell Prompt
The fastest way to customize your shell prompt without any dependencies is to write your own prompt in your profile by modifying the `prompt` function:

```PowerShell
# Custom shell prompt for Windows PowerShell profile
function prompt {
    "[$(Get-Date -format 'hh:mm:ss tt')] PS $($executionContext.SessionState.Path.CurrentLocation)$('>' * ($nestedPromptLevel + 1)) ";
}
```

This will change your prompt to include the time and look more like this:

```example
[08:30:28] PS C:\Users\griff >
```

## Using OhMyPosh for Easy Customization
We can take care of customizing the theme and shell prompt simultaneously using the Oh My Posh prompt theme engine. 

You can install OhMyPosh several ways:
- Run `winget install JanDeDobbeleer.OhMyPosh -s winget` in PowerShell
- If you use the Scoop package manager, you can run `scoop install https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/oh-my-posh.json` in PowerShell
- Install from the Microsoft Store

Once you have it installed, you can select a theme from their [themes page]((https://ohmyposh.dev/docs/themes). Once you've picked a theme, you'll want to edit your PowerShell profile to utilize the theme. You can point the `oh-my-posh` command at the URL of the theme's configuration, or you can download that configuration and point `oh-my-posh` at the local file.

Using local path:
```PowerShell
oh-my-posh init pwsh --config 'C:\Users\griff\jandedobbeleer.omp.json' | Invoke-Expression
```

Using online URL:
```PowerShell
oh-my-posh init pwsh --config 'https://raw.githubusercontent.com/JanDeDobbeleer/oh-my-posh/main/themes/jandedobbeleer.omp.json' | Invoke-Expression
```

If you want to download an online theme locally, you can use this:

```PowerShell
Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/JanDeDobbeleer/oh-my-posh/main/themes/jandedobbeleer.omp.json' -OutFile 'C:\Users\griff\jandedobbeleer.omp.json'
```

And then you can use that local path in your configuration. Once you've added one of the above commands to your profile, save it and close/re-open your terminal, and you should see your newly themed shell prompt!

Be sure to check out the Oh My Posh documentation which is linked in the Additional Resources section of this blog post to learn about getting started in detail, and see all of the neat things you can do with it!

I'm using the Pure theme in the screenshots at the beginning of this post.

# Add Aliases and Functions
You can add custom aliases to your PowerShell profile as well. I have these in place since I use NeoVim as a terminal-based text editor, but reflexively use the `vi` and `vim` commands even on Windows:

```PowerShell
Set-Alias -Name 'vi' -Value 'nvim'
Set-Alias -Name 'vim' -Value 'nvim'
```

This means that I can use `vi` or `vim` to edit files in-terminal instead of having to remember to use `nvim` for NeoVim; this is a more comfortable experience from me since `vim` is what I use on Linux.

Additionally, you can add custom functions to your profile. Here's a fun one that uses the **wttr.in** GitHub project to get the weather (be sure to check out the project in the Additional Resources section at the end of this post):

```PowerShell
function Get-Weather {
	Invoke-RestMethod -Method GET -Uri 'https://wttr.in/cupertino?format=3'
}
```

With that addition, you can use `Get-Weather` to get your specified location's current weather forecast:

```output
cupertino: ☀️   +48°F
```

While this particular example might seem trivial, hopefully you can see just how helpful and flexible this can be!

I've also added functions specific to my work environment, such as commands that let me interact with incidents, changes, and other items in our ITIL service management platform, quick scripts for checking disk space, unlocking Active Directory users, getting logs, etc. I'll leave those up to your imagination based on your needs in your environment.

# Adding an in-terminal Text Editor
As someone who spends a fair bit of time administering Linux servers from a command line, one thing that I can't live without is the ability to quickly edit files from the terminal without launching a GUI-based application. Naturally, I had to install a terminal-based text editor! 

You can install [Nano for Windows](https://github.com/okibcn/nano-for-windows) or [Vim](https://www.vim.org/download.php) or other editors if you so desire. 

I personally use [NeoVim](https://neovim.io/) and have the [NVChad](https://github.com/NvChad/NvChad) configuration installed as shown in the screenshots at the beginning of this post.

# Message of the Day
Linux admins will be familiar with the MOTD concept. If we want to specify a message of the day in our profile, we can simply `Write-Host` with the message:

```PowerShell
Write-Host "Welcome to your customized profile!"
```

If you want to go way overboard, you can execute applications and have them output into the terminal as part of the message of the day as well.

In the screenshots at the beginning of this post, you'll see system information printed in the terminal. This is due to my having [Winfetch](https://github.com/lptstr/winfetch) (a Windows alternative to Neofetch on Linux systems) installed, and calling it in my profile script:

```PowerShell
winfetch
```

# Working with Multiple Profiles
You might want to use multiple PowerShell profiles for different scenarios or purposes. For example, you might have a profile for the PowerShell console and another one for VS Code. Or you might have a profile for your personal use and another one for your work use.

Earlier we talked about using `$profile` to print the path to your current profile to the terminal. When working with multiple profiles, you can use `$profile.CurrentUserAllHosts` to return the path of the profile for the current user and all hosts.

If you want to create a new profile for all users on the current host, you can do:

```PowerShell
New-Item -Path $profile.AllUsersCurrentHost -ItemType File -Force
```

To load a different profile than the current one, you can use the dot sourcing operator followed by the path to the profile file:

```PowerShell
. $env:userprofile\Documents\other_profile.ps1
```

# Performance Considerations
There are a lot of cool things we can do in our profile script, **but** it is important to keep in mind that the more you put into your profile script, the more time it takes for your profile to load when you start your terminal emulator. You'll want to avoid pre-loading entire modules if you don't need to, and also limiting the number and length of functions in your profile. If needed, reference another script in your profile instead of including the entire script. Keep these types of things in mind as you customize your profile to ensure you don't spend needless seconds (or worse, minutes) waiting for your profile to load.

# Conclusion
In this post, we have learned how to use PowerShell profiles to customize our PowerShell environment and enhance our productivity. We have seen how to locate and edit the profile file, how to change the prompt theme using manual settings or Oh My Posh, how to add custom aliases and functions, how to install a terminal-based text editor, and how to work with multiple profiles for different scenarios or purposes. PowerShell profiles are a powerful and flexible way to tailor PowerShell to our needs and preferences. By using them wisely, we can make our PowerShell experience more enjoyable and efficient. I hope that you found some bit of this blog post helpful!

# Additional Resources
Theme Tools:
- [Oh My Posh Documentation](https://ohmyposh.dev/docs/)
- [ColorTool Documentation](https://github.com/microsoft/terminal/tree/main/src/tools/ColorTool)
  
Terminal-based Text Editors:
- [NeoVim Documentation](https://neovim.io/doc/)
- [NVChad Documentation](https://github.com/NvChad/NvChad)
  
Fun Tools:
- [Wttr.in - Check the Weather from your terminal](https://github.com/chubin/wttr.in)
- [Winfetch - Print pretty system information to your terminal](https://github.com/lptstr/winfetch)
  
PowerShell Profiles Documentation:
- [about Profiles - PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-7.3)