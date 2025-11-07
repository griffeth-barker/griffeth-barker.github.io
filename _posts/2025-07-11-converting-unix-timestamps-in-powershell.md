---
title: "Converting Unix Timestamps in PowerShell"
tags:
- powershell
- windows
- unix
- linux
- datetime
---

# Converting Unix timestamps in PowerShell

## The Need
This will be a quick one. While working with a platform for which I am writing a PowerShell module, I began to frequently interact with Unix timestamps, and needed to be able to convert them to workable DateTime objects. While Microsoft.PowerShell.Utility includes a variety of helpful cmdlets to conver between data formats, it does not offer a way to convert between these two ways of expressing a datetime.

## A Solution
In an attempt to make my life easier during the development of this little PowerShell module, I knocked up these quick and dirty functions for use (not in the module, but in my personal PowerShell environment in general).

### ConvertFrom-UnixTimestamp
```PowerShell
function ConvertFrom-UnixTimestamp {
    <#
    .SYNOPSIS
        Converts a Unix timestamp to a DateTime object.
    .DESCRIPTION
        Converts a Unix timestamp to a DateTime object.
    .PARAMETER Milliseconds
        The Unix timestamp to convert.
    .INPUTS
        String
    .OUTPUTS
        DateTime
    .EXAMPLE
        ConvertFrom-UnixTimestamp -Milliseconds 1747927975516
        # Converts the Unix timestamp to a DateTime object.
        # Example output: 
        #   Thursday, May 22, 2025 8:32:55 AM
    #>

    param(
        [Parameter(Mandatory = $true, ValueFromPipeline = $true)]
        [string]$Milliseconds
    )

    begin {}

    process {
        $output = ( [DateTime]::UnixEpoch ).AddMilliseconds( $Milliseconds ).ToLocalTime()
        Write-Output $output
    }

    end {}
}
```

### ConvertTo-UnixTimestamp
```PowerShell
function ConvertTo-UnixTimestamp {
    <#
    .SYNOPSIS
        Converts a DateTime object to a Unix timestamp.
    .DESCRIPTION
        Converts a DateTime object to a Unix timestamp.
    .PARAMETER DateTime
        The DateTime object to convert.
    .INPUTS
        DateTime
    .OUTPUTS
        String
    .EXAMPLE
        ConvertTo-UnixTimestamp -DateTime (Get-Date)
        # Converts the DateTime object for today's date to a Unix timestamp.
        # Example output: 
        #   1752252878823
    #>

    param(
        [Parameter(Mandatory = $true, ValueFromPipeline = $true)]
        [DateTime]$DateTime
    )

    begin {}

    process {
        $output = [math]::Round( ($DateTime.ToUniversalTime() - [DateTime]::UnixEpoch).TotalMilliseconds )
        Write-Output $output
    }

    end {}
}
```

Both functions are available in this [GitHub Gist](https://gist.github.com/griffeth-barker/fea781e93fb5d6f73b7821b207c8747a).

## Wrapping up
With PowerShell Core having a focus on being cross-platform, it would seem logical to me that such functionality could be helpful to have. Who knows? Perhaps some day we'll get some similar functionality.

If you found this helpful, I'd appreciate it if you starred the GitHub Gist. Thank you!
