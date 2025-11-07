---
title: "Keeping software up to date using Chocolatey"
tags:
  - windows
  - software
  - patching
  - chocolatey
  - powershell
---

# Introduction

Don't you love those pop-ups you get in the corner of your desktop telling you there's an update for a program installed on your computer? Or how about when you open a program and it tells you there's an update? You just want to use your software, so if you're like me, you've probably clicked "ignore" or closed the update to just get to what you were doing. But there is a better way! Nobody wants to take the time to keep all of their software up-to-date. In this brief blog post, I take a look at how you can make this happen "auto-magically," at least for many common programs.

# Installing Chocolatey package manager

![](https://audiomgtmoregame.files.wordpress.com/2020/12/chocolatey-logo.png?w=1024)

The first thing we will need to do is install the Chocolatey package manager. This is going to let us have access to one central source for the programs and their updates.

To install Chocolatey, open Windows PowerShell by going to the Start Menu and typing `powershell` . Right-click the top result, and click "Run as Administrator." This will require you to have administrative rights. If you do not have administrative rights, this process will not be possible.

In the Windows PowerShell window that appears, paste the following command:
```PowerShell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

Once that command has completed, close Windows PowerShell and launch it again with administrative rights like we just did.

# Installing packages

Now we need to install the packages that we want to keep up-to-date. To find out if a program you have has a package available, use the below example in PowerShell:

choco search _programname_

where _programname_ is the name of the program you want. Some examples might be:

- adobereader
- adobe-connect
- googlechrome
- microsoft-edge
- discord
- putty
- icue
- bitwarden
- lastpass
- obs-studio
- conemu
- forticlientvpn
- openvpn
- java
- jre
- logitech-options
- reflect-free
- and many, many more.

To actually install any or a combination of these, use the following command:
```PowerShell
choco install programname -y
```

where `programname` is the package name. You can list multiple package names in succession to install multiple programs at once:

```PowerShell
choco install programname1 programname2 programname3 -y
```

Chocolatey will download the packages and install them.

# Writing the update script

With the packages installed, now we need to create a super simple script that will update those packages.

Open Notepad by going to the Start Menu and typing "notepad" (quotations omitted) and running the top result.

In the blank document that opens, type the following:
```PowerShell
choco upgrade all -Y
```

If you continually run into errors with certain packages updating and don't mind the insecurity of it, you can also use:

choco upgrade all -Y --ignore-checksums

>[!warning] Warning
>This is not necessarily a security best practice. You should proceed with caution and as appropriate to your environment.

Now save this by going to File > Save As... In the Save As window, change the drop-down menu for the file type. It will be defaulted to .txt and we want to change it to All Files. Type a name for the file ending with .ps1. Example:

chocolatey-updater.ps1

Save it to a location where it won't be touched and will always be available. I have mine saved to a folder I created at C:\ScheduledTasks but you can put yours wherever you want.

## Scheduling automatic updates

Finally, now that we have installed packages and created a script to update them, we will want to schedule the update process to be completely automatic. Because being hands-off is the whole point!

Open the Task Scheduler by clicking the Start Menu and typing "task scheduler" then running the top result.

In the resulting Task Scheduler window, open the Action menu, then click Create Task.

![](https://audiomgtmoregame.files.wordpress.com/2020/12/screenshot-2020-12-15-135956.jpg?w=532)

Give your scheduled task a name and a description, then select "Run whether user is logged in or not" and check "Run with highest privileges." Finally, Configure for: Windows 10.

Move to the Triggers tab.

![](https://audiomgtmoregame.files.wordpress.com/2020/12/image-3.png?w=976)

Add a new trigger to begin the task on a schedule. Input your desired start date, reoccurence period, and ensure "Enabled" is checked, then click OK.

Move to the Actions tab.

![](https://audiomgtmoregame.files.wordpress.com/2020/12/image-4.png?w=972)

Make the action "Start a program" and in the Program/script field, paste the path to PowerShell:

%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe

In the Add arguments field, add the following:

-file scriptpath

where scriptpath is the path to where you saved your script earlier. My example looked like this:

-file C:\ScheduledTasks\chocolatey-updates.ps1

When done, click OK on all remaining Task Scheduler windows and the task will be scheduled. Now, as long as your computer is on, you leave the task scheduled, and the script is available where you saved it, your computer should automatically update your specified programs without you having to do anything at all.

I have been keeping my computers up to date like this for several years (set to automatically update everything weekly) and it saves me tons of time, and I rarely get notifications about new versions or annoying popups anymore.

## Bringing it all together

In this process, we installed Chocolately package manager, figured out what programs we wanted to keep up to date that were available through Chocolatey, installed those programs, wrote the update script/command, and scheduled PowerShell to run that command at a regular interval.

How did this go for you? Got any other neat tips and tricks for keeping your system running in great shape? I'd love for you to share them in the comments.