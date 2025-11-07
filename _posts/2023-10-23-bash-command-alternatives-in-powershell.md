---
title: Bash Command Alternatives in PowerShell
tags:
  - terminal
  - powershell
  - bash
---

Bash and PowerShell are two popular command-line shells that can help you automate tasks and manage your system. Bash is the default shell for Linux and macOS, while PowerShell is the default shell for Windows. If you are an IT professional who has some experience in using Bash, but need to administer a Windows environment, this might just be helpful to you. In this post, we'll take a look at some common Bash commands and their equivalent (more or less) commands in PowerShell.
# General Commands
Below is a quick cheat sheet of common Bash commands and how to accomplish the same thing in PowerShell. Many of the commands are pre-aliased in PowerShell and will work as you'd expect, while others require use of different commands to accomplish the desired outcome.

| Bash      | Bash Command works in PoSh | PowerShell Cmdlet   | PowerShell Alias | Purpose                                          | Comment                                                                                                                                                                                     |
| --------- | -------------------------- | ------------------- | ---------------- | ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cat`     | No                         | `Get-Content`       | `gc`             | Read content from a file.                        |                                                                                                                                                                                             |
| `clear`   | Yes                        | `Clear-Host`        | `clear`          | Clear the terminal.                              |                                                                                                                                                                                             |
| `cp`      | Yes                        | `Copy-Item`         | `copy`           | Copy an item.                                    |                                                                                                                                                                                             |
| `curl`    | Yes                        | `Invoke-WebRequest` | `iwr`            | Request data from a URL.                         | To truly match the default `wget` behavior, you'll need to wrap the command in parenthesis then use the `.Content` dot operator, or pipe the command's output into `Select-Object Content`. |
| `diff`    | Yes                        | `Compare-Object`    | `diff`           | Compare content of two objects.                  |                                                                                                                                                                                             |
| `grep`    | No                         | `Select-String`     | `sls`            | Find a matching string.                          | To truly match the default `grep` behavior, you'll need the `-Pattern` and `-SimpleMatch` parameters.                                                                                       |
| `history` | Yes                        | `Get-History`       | `h`              | Show previous commands executed at the terminal. |                                                                                                                                                                                             |
| `kill`    | Yes                        | `Stop-Process`      | `kill`           | End a process.                                   |                                                                                                                                                                                             |
| `ls`      | Yes                        | `Get-ChildItem`     | `gci`            | List items in a directory.                       |                                                                                                                                                                                             |
| `man`     | Yes                        | `Get-Help`          | `man`            | View command help file.                          |                                                                                                                                                                                             |
| `mkdir`   | Yes                        | `mkdir`             | `md`             | Create a directory.                              |                                                                                                                                                                                             |
| `mv`      | Yes                        | `Move-Item`         | `mi`             | Move an item.                                    |                                                                                                                                                                                             |
| `ps`      | Yes                        | `Get-Process`       | `ps`             | View a running process.                          |                                                                                                                                                                                             |
| `pwd`     | Yes                        | `Get-Location`      | `gl`             | Show the present working directory.              |                                                                                                                                                                                             |
| `rm`      | Yes                        | `Remove-Item`       | `rm`             | Remove an item.                                  |                                                                                                                                                                                             |
| `tail`    | No                         | `Get-Content`       | `gc`             | Read the contents of a file.                     | To truly match the default `tail` behavior, you'll need the `-Wait` parameter.                                                                                                              |
| `tee`     | Yes                        | `Tee-Object`        | `tee`            | Redirect output to two locations.                |                                                                                                                                                                                             |
| `wget`    | Yes                        | `Invoke-WebRequest` | `iwr`            | Request data from a URL.                         | To truly match the default `wget` behavior, you'll need to wrap the command in parenthesis then use the `.Content` dot operator, or pipe the command's output into `Select-Object Content`. |

It's important to note that while both the PowerShell Cmdlet or Alias can be used functionally, when writing scripts, you should always use the full Cmdlet name instead of the alias.

# Editing Files
Systems Administrators and other IT professionals who have worked with Linux before will know the convenience of being able to edit a file directly in the terminal using text editors such as `vim` or `nano` or others. These two editors in particular are nice because they come pre-packaged with many Linux distributions.

Unfortunately, Windows does not have such functionality in PowerShell. Luckily, we can install both Vim and Nano on Windows. While Windows does have a newly inbuilt package manager, `winget`, I still prefer `chocolatey` for its ease of use, reliability, and wide selection of available software.

If you don't have Chocolatey and want to use it to install these editors, you can install using this command as of the time of this post's writing:

```PowerShell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

> Note: You can check out the [Chocolatey website]([Chocolatey Software | Installing Chocolatey](https://chocolatey.org/install)) to verify if this is still the current install script.

With that installed (and after restarting your terminal), you can install both these editors:

```Powershell
# Install Nano text editor
choco install -y nano

# Install Vim text editor
choco install -y vim
```

You'll want to restart your terminal again since this will modify your profile, but after that you're all set to edit a file like you usually would in Bash:

```PowerShell
nano C:\Path\To\Your\File
```

I prefer Vim, and even once the package is installed, you'll need to use `vim` as your editor command unless you want to additionally create an alias for it:

```PowerShell
New-Alias -Name "vi" -Value "vim"
```

Now you can use Vim to edit files in the terminal to your heart's content:

```PowerShell
vi c:\Path\To\Your\File
```

# Conclusion
In this post we looked at some common Bash commands and their equivalents in PowerShell, as well as how to unlock the power of in-terminal text editing in PowerShell just like we have in Bash. While administering a Windows environment can be quite different from a Linux environment, there are many niceties that you can still take advantage of to make life easier. I definitely encourage any IT professional to learn more about PowerShell and its capabilities! You can find more resources on PowerShell on the [Microsoft Learn website](https://learn.microsoft.com/en-us/search/?terms=powershell). Keep those servers humming!