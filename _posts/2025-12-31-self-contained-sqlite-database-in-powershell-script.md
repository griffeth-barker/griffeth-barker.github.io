---
title: "A Database Engine, a Database, and a PowerShell Script Walk Into a Single File"
tags:
  - powershell
  - sqlite
  - ntfs
  - alternate-data-streams
  - database
categories:
  - powershell
---

You ever stumble onto some interesting information that gives you an idea? While browsing some documentation the other day, I learned about NTFS Alternate Data Streams and how you can interact with them in PowerShell. There are plenty of people well-versed in this but it was new to me. I successfully did a basic interaction test:

```powershell
$file = $env:USERPROFILE\Desktop\test.txt
Set-Content -Path $file -Value "Hello"
Set-Content -Path $file -Value "Goodbye" -Stream TestStream

Get-Item -Path $file -Stream *
# Output shows both the default :$DATA stream and our custom TestStream
```

Interesting! So we can store and access text very easily. So my next thought was, what else can we put in an alternate data stream? I decided to start working with a `.ps1` file, simply because I figured it might be useful later and I'm prone to creating PowerShell scripts.

> ⚠️  **WARNING**: This is for educational purposes only. It should not be considered production-ready, best-practice, etc. You should fully understand code before you run it on your system, and you should have authorization to run code on your system. The contents of this post and associated repository may trigger endpoint protection and antivirus, though the contents as published are not malicious. This little experiment uses SQLite, which is not affliated with me nor this project, and can be obtained from their [website](https://sqlite.org/index.html).

## Storing an application
For whatever reason, I opted to shove an application into an ADS of my PowerShell script file. So I made an attempt at encoding it. Turns out the Windows clipboard and some text editors really don't like when you have a few million characters in a single copy/paste action. I opted to have a build script that would generate the actual script that would contain the SQLite executable, which would avoid me needing to copy and paste the massive Base64-encoded string.

So now I had `build.ps1`, and it was successfully storing `sqlite3.exe` as a Base64-encoded string:
```powershell
    $exePath      = "$($env:USERPROFILE)\Desktop\sqlite3.exe"
    $targetScript = "$($env:USERPROFILE)\Desktop\main.ps1"
    $fileBytes    = [System.IO.File]::ReadAllBytes($exePath)
    $base64       = [Convert]::ToBase64String($fileBytes)
```
Neat, there's an application within the script file now. Now, how to actually execute it?

## Interface
I realized that I'd need a user interface for this, so that became the function of `main.ps1` in it's default data stream. 

I'd need to decode the stored `sqlite3.exe` binary so it could be run:

```powershell
    $UniqueId = [guid]::NewGuid().Guid
    $TempExe  = Join-Path $env:TEMP "sqlite_$($UniqueId).exe"
    $TempDb   = Join-Path $env:TEMP "db_$($UniqueId).db"

    $Bytes = [Convert]::FromBase64String($SQLiteBase64)
    [System.IO.File]::WriteAllBytes($TempExe, $Bytes)
    $dbBytes = [Win32Ghost]::ReadAds($CurrentScript + ":Data")
    if ( $dbBytes.Length -gt 0 ) {
        [System.IO.File]::WriteAllBytes($TempDb, $dbBytes)
    }
```

To keep the footprint clean, the script generates a unique GUID for the engine every time it runs. It exists for less than a second—just long enough to process the query—before a finally block wipes it from the disk. My original hope was to have this run entirely in memory, but I realized this is beyond my know-how, so for now I'm sticking with writing to disk temporarily and cleaning up post-query.

```powershell
    if ( Test-Path $TempExe ) { Remove-Item $TempExe -Force -ErrorAction SilentlyContinue }
    if ( Test-Path $TempDb ) { Remove-Item $TempDb -Force -ErrorAction SilentlyContinue }
```

A function within the file would be necessary, so `Invoke-SQLiteQuery` came into the picture. Initially, it just did a simple query against the database but I quickly realized I'd need to ensure the table existed. So why not just have the table created if it doesn't already exist? I added some validation for the ended up with something like this:

This is where I hit the biggest wall. As far as I can tell, Windows places a mandatory lock on a script while it's running. PowerShell cmdlets appear to respect this lock so strictly they won't let you write to the file's own hidden streams while the code is executing.

To bypass this, I had Google Gemini help me write a bridge to the Windows Kernel using C# (at least by my understanding). kernel32.dll!CreateFile API is used, which allows you to request permissive sharing bits that standard PowerShell simply cannot access.

```C#
public class Win32Ghost {
    [DllImport("kernel32.dll", SetLastError = true, CharSet = CharSet.Auto)]
    public static extern SafeFileHandle CreateFile(
        string lpFileName, uint dwDesiredAccess, uint dwShareMode,
        IntPtr lpSecurityAttributes, uint dwCreationDisposition,
        uint dwFlagsAndAttributes, IntPtr hTemplateFile);
    // ... Read/Write methods using the handle ...
}
```

This created a handoff point where PowerShell provides the path, and the Win32 API handles the "illegal" I/O. I do not know C#, and don't fully understand how exactly this works, but as far as I can tell, it lets PowerShell provide the file path then we can open a handle to the file using (probably overly) permissive access flags that I couldn't figure out how to do in PowerShell. **This is probably bad practice and/or unsafe.** But I was just playing around in a virtual machine so stakes were low.

## Using the Database
With a somewhat complete `build.ps1` script, I was able to successfully create `main.ps1` that stands on its own. No configuration, no external dependencies. All you have to do is start writing and reading records! The first time  you do this, the database engine will realize that the database needs to be initialized and handles it on-the-fly.

```powershell
# Insert operations
.\main.ps1 -Query "INSERT INTO TableName (Column1, Column2) VALUES ('Value1', 'Value2');"
# No output

# Select operations
.\main.ps1 -Query "SELECT * FROM TableName"
# Output:
# ID  Col1    Col2    CreationTime
# --  ----    ----    ------------
# 1   Value1  Value2  12/31/2025 6:53:05 PM
```

## Output
Speaking of classes earlier, I realized that when I would query the database, the object that was returned looked like a table, but was actually just a string. Being a fan of the PowerShell pipeline and objects, that simply wouldn't do, so I opted for a `[DatabaseRecord]` class:

```powershell
class DatabaseRecord {
    hidden [datetime]$_RetrievedAt
    DatabaseRecord([hashtable]$row) {
        $this._RetrievedAt = [datetime]::Now
        if ( $row.ContainsKey('ID') ) {
            Add-Member -InputObject $this -NotePropertyName 'ID' -NotePropertyValue ([int32]$row['ID']) 
        }
        foreach ($key in $row.Keys) { 
            if ($key -notin @('ID', 'CreatedAt')) { 
                Add-Member -InputObject $this -NotePropertyName $key -NotePropertyValue $row[$key] 
            } 
        }
        if ($row.ContainsKey('CreatedAt')) { 
            Add-Member -InputObject $this -NotePropertyName 'CreatedAt' -NotePropertyValue ([datetime]::Parse($row['CreatedAt'])) 
        }
    }
}
```

With that, my output was looking pretty good. The two metadata properties ID and CreationTime have their respective integer and datetime types, while any other SQL columns are returned as strings. You can expand selections on the returned object(s) and pipe them into PowerShell cmdlets/functions.

## Final Thoughts
This was a fun rabbit hole to fall down unexpectedly. I love finding weird things you can do with PowerShell and never hesitate to poke at them when I find them. Because of that, I now have a single file that has:
  1. A database engine
  2. A database
  3. A PowerShell script to interact with the database

🔗 **You can find the repository with the script on [GitHub](https://github.com/griffeth-barker/ADSSQLite).**

Does this solution support complex database operations? Almost certainly not. Is this a good solution for something? Unlikely. Is it secure and best-practice? Also no! But was it interesting and fun? **Yes.**
