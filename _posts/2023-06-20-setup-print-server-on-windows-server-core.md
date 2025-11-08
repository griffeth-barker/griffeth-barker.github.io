---
title: "Setup a print server using Widnows Server Core"
tags:
  - windows-server
  - powershell
categories:
  - powershell
---

# Introduction

Many administrators are always looking for ways to simplify the required management of their environments, save on overhead, resources, etc. With the advent of Windows PowerShell and it growing to be a commonplace administrator utility in Windows environments, simple systems such as print servers no longer really require a full-fledged graphical user interface. Print servers are prime candidates to be hosted on the CLI-based Windows Server Core variant of Windows Server, especially where they are so easily managed through remote PowerShell. In this brief article, we'll take a quick look at spinning up a core print server and some of the options available for administering the new server.

# Spinning up your machine

Whether you are using traditional hardware for your server (a rackmount server, blade server, tower server/desktop PC) or a virtualized environment, you'll first need to actually install Windows Server Core on your machine. For my environment, I spun up a virtual machine in my Proxmox virtual environment, but many may be using VMware, Hyper-V, or other solutions which will work perfectly fine.

# Adding the features and roles to the server

Once you have Windows installed and your requisite network and domain configuration done, add the necessary services and features to the server:

```PowerShell
Install-WindowsFeature Print-Services
```

This will install the Print Services role as well as the Print Server Role Service. No further configuration is necessary as far as the services and features go. All that is left to do is add some printers!

# Administering the server

There are multiple ways to administer your new Windows Core print server.

### PowerShell

Arguably the correct way to manage a Windows Core server, is PowerShell. Like other deployments, a print server is managable through the command line with general ease. Beginning with Server 2012's Core version, there are many print management commands available to administrators.

As an example, we can configure a printer using just two commands:

Adding the printer port:

```PowerShell
Add-PrinterPort -Name "192.168.254.5" -PrinterHostAddress "192.168.254.5"
```

Adding, sharing, and publishing the printer:
```PowerShell
Add-Printer -Name YourPrinter01 -DriverName "HP Universal Print Driver PCL6" -PortName 192.168.254.5 -Shared -ShareName "YourPrinter01" -Published
```

### VBS

An archaic option also exists. At `C:\Windows\System32\Printing_Admin_Scripts` you'll find a variety of VBS scripts that can be used to administer the print server. But seriously, who is preferring VBS when we have PowerShell?

As an example, we can configure a printer using the following commands:


```PowerShell
# Creating the printer port:
cscript prnport.vbs -a -r 192.168.254.5 -h 192.168.254.5 -o raw

# Adding printer on a specific printer port:
cscript prnmngr.vbs -a -p YourPrinter01 -m "HP Universal Print Driver PCL6" -r 192.168.254.5

# Sharing the printer:
cscript prncnfg.vbs -t -p YourPrinter01 -r 192.168.254.5 -h YourPrinter01 +shared -direct -m "Default printer for HR" -l "YourDesiredLocation"

# Publishing the printer to Active Directory:
cscript pubprn.vbs \\printserver\YourPrinter01 "LDAP://CN=YourContainer,DC=YourDomain,DC=com"
```


### GUI

If you really just can't let go of using the graphical user interface just yet as you ease into command-line-based administration you can still get your hands on the GUI by executing: `C:\Windows\System32\printui.exe /il`

Additionally, on a remote machine, you can connect RSAT to your Windows Core print server and use the Print Management snap-in for the Microsoft Management Console to visually administer the print server.

But seriously...get comfortable with PowerShell. This is the way.

# Conclusion

And, well, that's pretty much it. There is certainly a more granular level of detail we could go into, but for the purposes of a general overview, I think that does it. Windows Core servers are quick and easy to spin up and an absolute breeze to configure if you are comfortable with PowerShell. In a lab environment, they can also be a great way to get more comfortable with PowerShell if you are just learning. Go ahead and spin up a Windows Core server in VirtualBox or your choice of virtualization bench and give it a try!

What are you using core servers for in your environment, and how do you like it? I'd love to hear from you in the comments below.

---

## Additional Reading

[Install Print and Document Services | Microsoft Docs](https://www.blogger.com/blog/post/edit/8562458718891840214/175097051257976925#)

[Using Server Core as a Print Server - Microsoft Tech Community](https://www.blogger.com/blog/post/edit/8562458718891840214/175097051257976925#)