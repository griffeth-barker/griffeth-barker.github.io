---
title: Fix Last Modified Dates on Notes when Migrating to Obsidian
tags:
  - obsidian
  - powershell
---
In this quick blog post, we'll take a look at a problem that frequently plagues people who are migrating to Obsidian from another notetaking platform.

> Warning: Remember that you should always fully understand what a command or script does before running it on any computer. This fix is not associated with Obsidian and its developers, nor is it guaranteed or warrantied in any way. Use at your own risk, and always have a backup!
# Problem
When you export all of your notes from the platform you're leaving, the **Last Modified** datetime for all of the notes will appear as the date you exported them from the other notes platform. While that's technically true, we'll want to make sure that this gets fixed to reflect the *actual* datetime that the note(s) were last modified.

# Prerequisites
This fix will require that all of your notes contain Properties in their YAML frontmatter. The main property that we are concerned with is the `Last Update` property.

# Solution
With a little bit of help from PowerShell, we can easily resolve this issue using a script that I threw together after a conversation with some other Obsidian users in the [Obsidian Members Group Discord Server](https://discord.gg/obsidianmd). 

The first thing we need to do is enumerate all of the notes in your Obsidian Vault:

```PowerShell
$notes = @(Get-ChildItem -Path C:\path\to\your\vault\*.md" -Recurse | Select-Object -ExpandProperty FullName)
```

Next, we need to loop/iterate through all of those notes, read the content of the note to get the `Last Update` property from the YAML frontmatter of the note, and then set the `LastWriteTime` property of the note file to that value:
```PowerShell
foreach ($note in $notes) {
    $last_write_time = (get-content -path "$note" | Select-String "Last Update" -SimpleMatch | Out-String).split(" ")[2]

    (Get-Item -Path "$note").LastWriteTime = $last_write_time
}
```

# Get the Script
You can download the script for your own free use by:
1. Right-click your Start Menu
2. Select Windows PowerShell
3. Copy/Paste this command into PowerShell and press ENTER:

```PowerShell
Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/griffeth-barker/public/main/obsidian/fix_last_modified_dates.ps1' -OutFile "$($env:userprofile\Downloads\fix_last_modified_dates.ps1)"
```

Once you've downloaded the script, open it in Notepad:

```PowerShell
Set-Location -Path "$($env:userprofile\Downloads)" ; notepad.exe .\fix_last_modified_dates.ps1
```

Change the `$vault` value to reflect the file path to the root folder of the Obsidian Vault, then save and close it.

Go back to PowerShell, and do `.\fix_last_modified_dates.ps1` and press ENTER.

# Conclusion
This script should be able to assist you with fixing the **Last Modified** date that shows up in File Explorer for all your notes in your Obsidian Vault that were previously exported, which resulted in all their dates being the same. I hope that this was helpful to you!
