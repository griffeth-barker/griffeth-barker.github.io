# Downloading Files Using PowerShell
After [responding to a post on BlueSky](https://bsky.app/profile/griff.systems/post/3ltyxrxhb5c2q) this morning, I realized that I have not yet written about methods of downloading files in PowerShell. We download things in a graphical user interface all the time, but sometimes you have the need to download a file as part of a script, automation, etc. Let's have a quick look at some methods of downloading files in PowerShell. Let's use the extended Alpine Linux ISO as our sample file to download.

# Methods
## Invoke-WebRequest
The first method that post PowerShellers will discover is the fact that `Invoke-WebRequest` has a `-OutFile` parameter. If you've used  `curl -O` before, this is not dissimilar from that behavior.
```PowerShell
# Note that the variable notes don't need to match the parameter names, even though I did as such here.
$uri = ""
$outFile = "$env:TEMP\"
Invoke-WebRequest -Uri $uri -OutFile $outFile
```
This will download the .zip archive to your user profile's temp directory. While downloading, you'll see a progress bar:
![]/content/published/media/invoke-webrequest-with-progress-bar.png

But how long did that take? Let's find out by wrapping our script inside `Measure-Command`:
```PowerShell
Measure-Command -Expression {
  $uri = ""
  $outFile = "$env:TEMP\"
  Invoke-WebRequest -Uri $uri -OutFile $outFile
}
```
As you may have noticed, `Invoke-WebRequest` is somewhat slow by default. It's more than sufficient for quickly downloading smaller items like configuration files and smaller compressed archives, but for larger files, it's not so fast:
![]

If you don't care about displaying the progress bar in the terminal, we can improve the performance by opting to not calculate and render it by changing your `$ProgressPreference` to "SilentlyContinue" instead of "Continue", which is the default, and using the same download script:
So what sort of performance improvement do we realize? Let's use the same measurement method again:
```PowerShell
Measure-Command -Expression {
  # Change the ProgressPreference first
  $ProgressPreference = 'SilentlyContinue'

  $uri = ""
  $outFile = "$env:TEMP\"
  Invoke-WebRequest -Uri $uri -OutFile $outFile

  # I recommend setting this back to the default value when done, so you don't get caught off guard in the future when expecting the default progress bar/output 😁
  $ProgressPreference = 'Continue'    
}
```
We see a decent performnace uplift here, having saved around 5 minutes!
![]

## System.Net.WebClient
Now if we really want to boost our performance when downloading files in PowerShell, we can make use of the `[System.Net.WebClient]` .NET class. PowerShell is build on .NET and can make use of such powerful classes. Let's try measuring this one now:
```PowerShell
Measure-Command -Expression {
  $uri = ""
  $outFile = "$env:TEMP\"
  $webClient = New-Object -TypeName System.Net.WebClient
  $webClient.DownloadFile("$uri","$outFile")
}
```
*Now* we're talking! Check out that performance uplift:
![]

# Wrapping Up
