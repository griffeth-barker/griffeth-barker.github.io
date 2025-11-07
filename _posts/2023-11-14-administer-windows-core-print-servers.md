---
title: "Administer Windows Core Print Servers"
tags:
  - powershell
  - print-services
  - windows-server
---
# Administer Windows Core Print Servers
You RDP to a print server and where you expected to see a Windows Server desktop environment, you are presented with a black screen with nothing more than a terminal window. How are you going to administer the print server?

Sure, you could launch the Print Management application from another server and then remotely administer the print server from there, and that's perfectly acceptable. But PowerShell can take care of this as well!

Here are some ways to complete common print server tasks using PowerShell in a Windows Core environment.

## Common Print Server Tasks
### Get a List of the Server's Printers
Need to see what printers are installed on the print server?

```PowerShell
Get-Printer
```

### Get a List of Print Jobs for a Specific Printer
Need to see what print jobs are pending for a printer on the server?

```PowerShell
Get-PrintJob -PrinterName printer01
```

### Get Printer Configuration
Need to check basic configurations such as if the printer is set to default to color printing, or double-sided printing (duplexing)?

```PowerShell
Get-PrintConfiguration -PrinterName printer01
```

### Get List of Installed Printer Drivers
```PowerShell
Get-PrinterDriver
```

### Get List of Configured Printer Ports
```PowerShell
Get-PrinterPort
```

### Remove a Print Job
Once you've gotten the print job in question using the above command and noted the ID of the print job, you can pipe that output into the `Remove-PrintJob` command:

```PowerShell
Get-PrintJob -ID 1 | Remove-PrintJob
```

### Restart the Print Spooler Service
A common task on print servers is to bounce (restart) the Print Spooler service. This can be accomplished like so:

```PowerShell
Restart-Service -Name spooler
```

## Conclusion
There are many other tasks you can perform for print servers hosted on Windows Core servers. Additionally, as with most PowerShell commands, you can invoke these commands remotely by using the `Invoke-Command` command and specifying the remote print server's name in the `-ComputerName` parameter.

Don't hesitate to use the terminal to perform these tasks! It can definitely be faster, especially as you get more comfortable with the command line.
