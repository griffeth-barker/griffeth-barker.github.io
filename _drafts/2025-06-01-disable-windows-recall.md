---
title: "Disable Windows Recall using PowerShell"
tags:
- powershell
- windows
- recall
---

# Disabling Windows Recall
If you’ve recently updated Windows or bought a new Windows computer, you may have noticed something called “Windows Recall” In this quick article, I’ll explain what Windows Recall does, why some users might want to disable it, and share a simple PowerShell function I wrote to make that easy.

## What Is Windows Recall?
Recall is a new feature Microsoft has introduced in certain builds of Windows. The idea behind it is to help you “recall” what you’ve been doing on your device. It works a bit like your photos timeline on your phone, but instead Recall captures screenshots of your activity, then performs some behind-the-scenes magic to recognize text and whatnot so you can search through them later. Think of it as a more advanced version of browser history, but for your whole desktop.

While this certainly sounds useful, not every user is comfortable with this.

## Why Disable Windows Recall?
There are several reasons you might not want this feature on your computer.

First up are the privacy concerns. Windows Recall captures regular snapshots of your screen. Even though Microsoft states this data stays local to your machine and is encrypted, some users still are not comfortable with the idea of their screen contents being logged behind the scenes. Further, the database which stores the Recall content is, last I checked, unencrypted once the user logs in.

Another reason some folks may not want Recall is that it runs in the background all the time and continuously collects data. Becauase of this, there might be some impact on system performance—especially on lower-spec hardware.

Finally, there are just some users prefer to keep their system as lean as possible and only run what they actually use (I totally get this). If you don’t plan on using Recall, removing it can be part of good piece of digital hygiene.

If while reading the above you found yourself nodding at any point, then the below disable function is for you!

## PowerShell Function to Disable Windows Recall

To make this easy, I wrote a PowerShell function that removes the feature cleanly. At the high level, all it does is use the `Disable-WindowsOptionalFeature` cmdlet to--well--disable the optional Windows feature for Recall. It is functionalized for portability and reusability.

The function is available in this [GitHub Gist]().

All you have to do is copy/paste the function into your elevated (run as Administrator) PowerShell terminal, then run `Disable-WindowsRecall`.

If you want it to run automatically after pasting it in without having to type the command, just uncomment the last line in the script before pasting into your terminal.

### Wrapping up
Windows Recall might be a great tool for some, but not everyone wants their system keeping such a detailed memory of what they've been doing. If you're the type who prefers privacy, minimalism, or jsimply maintaining as much control of your machine as possible, disabling Recall could be the move.

This portable PowerShell function enables you to simply copy/paste to disable this new feature introduced by Microsoft. If you want Recall back, you can re-enable it using `Enable-WindowsOptionalFeature -Name "Recall"`.

If you found this helpful, I'd appreciate it if you starred the GitHub Gist. Thank you!