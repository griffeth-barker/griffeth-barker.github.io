---
title: Search Log Files Using PowerShell
tags:
  - powershell
  - troubleshooting
  - logging
---
You're troubleshooting an application and need to look through the logs to find out what is happening. We've all been there, right? Log files are an important resource during troubleshooting. So much so, that if you open a ticket with the software vendor, their support will likely open up their communication by requesting the log files.

By reading and searching log files, you can find clues and insights that can help you diagnose and fix the problem. However, reading and searching log files can also be a tedious and time-consuming task, especially if you have to deal with large, complex, or multiple log files. That’s why using PowerShell to read and search log files can be a helpful time saver. PowerShell is a powerful scripting language that can automate and simplify many tasks related to system administration, including working with log files. 

In this blog post, we'll talk about how to use PowerShell to read and search log files during troubleshooting. 

Want to follow along? You can get the sample log file by using these commands:

```PowerShell
# Create a temp directory
$WorkingDir = "C:\temp"
if (!(Test-Path -Path $WorkingDir)){
  New-Item -Path "C:\" -Name "temp" -ItemType "directory" -Force
}
```

```PowerShell
# Set location to the temp directory
Set-Location -Path $WorkingDir
```

```PowerShell
# Get the sample log file
# LogHub on GitHub has a great sample log file that we'll use for this. Go check out their repository at https://github.com/logpai/loghub
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/logpai/loghub/master/Windows/Windows_2k.log" -OutFile "$WorkingDir\sample.log"
```

# Reading an Entire Log File

> If you're following along, be sure your terminal's Present Working Directory (pwd) is `C:\temp` and that you have the `C:\temp\sample.log` file in that directory.

The most basic task is getting the content of the log file. We can accomplish this simply:

```PowerShell
Get-Content sample.log
```

The above command will read the entire log file and display its contents in your terminal window; the output would look something like this (I've truncated the output to just the first item for the sake of reducing the amount of scrolling in this blog post, as the output is hundreds of lines long):

```output
2016-09-28 04:30:30, Info                  CBS    Loaded Servicing Stack v6.1.7601.23505 with Core: C:\Windows\winsxs\amd64_microsoft-windows-servicingstack_31bf3856ad364e35_6.1.7601.23505_none_681aa442f6fed7f0\cbscore.dll
...
```

Being able to read the file is one thing, but it's not the *most* helpful thing in the world.

# Searching a Single Log File
What if we have an error and want to find out more about it? We can search the log file for the error using the `Select-String` command:

```PowerShell
Select-String -Path .\sample.log -Pattern "Failed to upload all unsent reports." -SimpleMatch
```

This first gets all the content of the log file, then selects any lines that have our search pattern in them. The `-SimpleMatch` switch allows for a basic search instead of the default search which uses regular expressions (regex).

> Note: If you want to shorten up this command, you can replace `Select-String` with its alias `sls`.

Now the output looks like this:

```output
2016-09-28 04:30:31, Info                  CBS    SQM: Warning: Failed to upload all unsent reports. [HRESULT =
0x80004005 - E_FAIL]
2016-09-29 00:00:46, Info                  CBS    SQM: Warning: Failed to upload all unsent reports. [HRESULT =
0x80004005 - E_FAIL]
```

Now we'd know that some unset reports failed to upload, have their timestamps, and the error code which likely denotes the reason that they failed to upload (though we'd need to consult the application vendor's documentation to verify).

This same example could be used to look for any instances of a specific error message in the log:

```PowerShell
Select-String -Path .\sample.log -Pattern 0x80004005 -SimpleMatch
```

> Note: The `-Pattern` parameter can accept multiple items in an array as an input! You could search two errors by using `Pattern '0x800004005','0x800005007'` if you needed to.

What if we wanted to look at the first error we returned for the Failure message and get some context? `Select-String` has a parameter `-Context` that lets us return X number of lines before the matched pattern, and Y number of lines after the matched pattern. Let's get that first instance of the error and also get the 3 lines before the error to see if something else relevant to the issue happened:

```PowerShell
Select-String -Path .\sample.log -Pattern "Failed to upload all unsent reports." -SimpleMatch -Context 3,0 | Select-Object -First 1
```

In the above example, `Get-Content sample.log` is reading the entire log file, and then we pipe that output into `Select-String -Pattern "Failed to upload all unsent reports." -SimpleMatch -Context 3,0` where we search for the error using `-Pattern` and `-SimpleMatch`, and also ask for the 3 lines before the error with `-Context 3,0`.

The output looks like this:

```output
  2016-09-28 04:30:31, Info                  CBS    SQM: Failed to start upload with file pattern:
C:\Windows\servicing\sqm\*_std.sqm, flags: 0x2 [HRESULT = 0x80004005 - E_FAIL]
  2016-09-28 04:30:31, Info                  CBS    SQM: Failed to start standard sample upload. [HRESULT = 0x80004005
- E_FAIL]
  2016-09-28 04:30:31, Info                  CBS    SQM: Queued 0 file(s) for upload with pattern:
C:\Windows\servicing\sqm\*_all.sqm, flags: 0x6
> 2016-09-28 04:30:31, Info                  CBS    SQM: Warning: Failed to upload all unsent reports. [HRESULT =
0x80004005 - E_FAIL]
```

If this were a real-world situation, those preceding lines could tell us why the error occurred.

# Searching Multiple Log Files
Alright, that's not so bad for a single log file. But what if the application has multiple log files?
No problem! This gets a little more involved, but still easily doable. I'll provide an example here, but there's no follow-along since we only downloaded one log file for the sample.

By inserting a `*` into the path of the log file(s), we can account for any files in the folder which have the file extension `.log`. 

```PowerShell
Select-String -Path "C:\temp\*.log" -Pattern 'Failed' -List | Select-Object -Property Filename,LineNumber,Line | Format-Table -AutoSize
```

The output looks like this:

```output
Filename    LineNumber Line
--------    ---------- ----
sample.log          11 2016-09-28 04:30:31, Info                  CBS    SQM: Failed to start upload with file patte...
sample2.log          1 0000-00-00 00:00:00,  ERROR  The task failed successfully ;-)
```

> If you wanted, you could even output this into the PowerShell GridView, which will open the results in a GUI window that allows you to sort by columns and filter further. Use the same command but replace `Format-Table -AutoSize` with `Out-GridView` in that case.

With the above command, you now have results of your search from **two** log files in a single view, rather than opening both files and manually searching files.

# Filtering by Date
When troubleshooting, it's often helpful to narrow the scope of time involved. When you're dealing with systems that use verbose logging, there can be a lot of content to search through. The method of filtering by date can vary from situation to situation based on how the application logs its actions, but for this specific example, we'll essentially read the lines of the log and search for a partial datetime:

```PowerShell
Select-String -Path "C:\temp\*.log" -Pattern '2016-09-28 04:' -List | Select-Object -Property Filename,LineNumber,Line | Format-Table -AutoSize
```

The above command will return any entries in any log files in `C:\temp` which contain occurred on September 28th, 2016 between 4:00 AM and 4:59 AM. This is an example of significantly narrowing the scope of the logs you are reviewing, and the output looks like this:

```output
Filename   LineNumber Line
--------   ---------- ----
sample.log          1 2016-09-28 04:30:30, Info                  CBS    Loaded Servicing Stack v6.1.7601.23505 with ...
```

> Again, you could instead pipe the result into `Out-GridView` instead of `Format-Table` if you so choose.

# Find Recently Written Log Files
Many applications will truncate their log files to prevent a single log file from growing too large. You've probably seen a log file directory before that has multiple files in it like:
- events01.log
- events02.log
- events03.log
- etc...

If you want to be sure you only look at log files that have been **recently written to**, then this is how to find those:

```Powershell
Get-ChildItem -Path 'C:\temp' | Where-Object {$_.Name -like "*.log*" -and $_.LastWriteTime -gt (Get-Date).AddHours(-1)}
```

This will return you a list of files with the `.log` file extension in the `C:\temp` directory whose `LastWriteTime` property is one hour ago or newer. You can modify as needed to narrow down the time period you want.

# Watch a Log File
Alright so you know the log file that you need to look at, but of course nobody can tell you  the last time the issue occurred. Like we often do in IT work, we can have the person experiencing the issue reproduce the issue. We'll want to be watching the log file *while*
this happens to be sure we see the issue get logged. There's an easy way to do this with PowerShell!

```PowerShell
Get-Content -Path 'C:\temp\sample.log' -Tail 5 -Wait
```

Here we are getting the last 5 lines of the log file, and waiting for new entries to appear. As they appear, they'll be displayed in the terminal.

> Note: If you want to shorten this command a bit, you can replace `Get-Content` with its alias: `type`.

# Conclusion
In this post we looked at how to use PowerShell to read and search log files for troubleshooting purposes. We have seen how to read an entire log file using the `Get-Content` cmdlet, how to search a single log file using the `Select-String` cmdlet, and how to search multiple log files at once using the `Get-ChildItem` and `Where-Object` cmdlets. We have also learned how to filter log files by date using the `LastWriteTime` property.

By using PowerShell to work with log files, we can save time and effort, as well as perform more advanced and flexible searches. PowerShell is a powerful scripting language that can help us with many tasks related to system administration, including log file analysis and troubleshooting.

I hope you enjoyed this blog post and found some bit of helpful information in it. The next time you need to review a log file (especially if its on a Windows Core server), do consider using these techniques to find the information you're looking for!
# Additional Resources
- [Get-Content (Microsoft.PowerShell.Management) - PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-content?view=powershell-7.3)
- [Select-String (Microsoft.PowerShell.Utility) - PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-string?view=powershell-7.3)
- [Get-ChildItem (Microsoft.PowerShell.Management) - PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-childitem?view=powershell-7.3)