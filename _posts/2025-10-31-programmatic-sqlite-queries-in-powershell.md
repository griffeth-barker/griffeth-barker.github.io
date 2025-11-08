---
title: "Programmatic SQLite Queries in PowerShell"
date: "2025-10-31"
tags:
  - powershell
  - sqlite
categories:
  - powershell
---

# Programmatic SQLite Queries in PowerShell
This will be a real quick one. After watching a short video on the history of SQLite and its applications, I decided to tinker with it a bit. It's a very easy database system to work with.
I found that the `sqlite3` binary accepts a string containing a command as an argument and realized that you can use this in conjunction with PowerShell to programmatically build and execute queries.

## Base Concept
I knew that you got into the SQLite command line by running `sqlite3` and then you could run SQLite queries and dot-commands from within the CLI. A quick bit of research told me that you can also specify a both a database file and a string after the binary name to run queries against the database without keeping the SQLite CLI open.
At it's foundation, its as simple as doing:
```powershell
sqlite3 ./path/to/database.db "SELECT * FROM EnablementReporting"
```
So where does this go from here? Here's my train of thought from the other day.

Can I pass the string in from the pipeline?
```powershell
"SELECT * FROM EnablementReporting" | sqlite3 ./path/to/database.db
```
Yep, that worked. Selecting information is pretty easy. 

What about doing inserts?
```powershell
sqlite3 ./path/to/database.db "INSERT INTO EnablementReporting ('Username','Status','DisabledDate') VALUES ('JohnSmith','Disabled','2025-10-30')"
```
That worked as expected, too. 

## Building Queries
Next up was programmatically building queries...
```powershell
$users = Get-ADUser -SearchScope "OU=Disabled Users,DC=internal,DC=griff,DC=systems"
foreach ( $user in $users ) {
    @(
        "INSERT",                     # Database operation
        "INTO EnablementReporting",   # Table to insert into
        "VALUES",                     # Indicates start of values container
        "(",                          # ---
        "'$($user.samAccountName)',", # Getting Username from AD user object's samAccountName property
        "'$($user.Enabled)',",        # Getting if user is enabled/disabled from AD user object's Enabled property
        "'$($user.whenChanged)'",     # Getting datetime of when the user was changed from the AD user object's whenChanged property
        ")"                           # ---
    ) -join '' | sqlite3 ./path/to/database.db
}
```
This also works! There's some goofiness with making sure you're including the necessary single-quotes and commas, but it works. And there's very likely a better way to do that. This was just a cursory discovery. So that's pretty much it. Nothing complicated and nothing inherently revelatory, but something I stumbled onto and found interesting. Maybe I'll have a cool use case for this some day. Maybe you do?

If you found this helpful or have a neat case where you're using PowerShell to build SQLite queries, I'd love to hear from you on BlueSky ([@griff.systems](https://griff.systems/bluesky)). Thank you!
