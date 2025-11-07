---
title: "Import PowerShell Modules Directly from GitHub or GitLab without Installing"
tags:
- powershell
- modules
---

# Import PowerShell Modules Directly from GitHub or GitLab without Installing
We know and love PowerShell modules. They do a lot of heavy lifting in the automation world! However, you might find yourself in a situation where you are not able to install a module on a system, but still want to make use of its functionality. For this purpose, I have `Import-ModuleFromGitx`.

This is a function that takes in the URL of a public, cloud-hosted, GitHub or GitLab repository and imports it directly into your current PowerShell session.

## Methodology
### Parameters
There are two parameters for the function.

`-Uri` is the URL of the repository you want to import. This required string can be piped in if you like, or specified directly.

`-OutputDir` is the target directory to which the module will be downloaded. This string is optional, and will default to the present working directory if not specified.

### URIs
The first problem I had is that the commonly used link for repositories is not a link that can be used to download the repository directly. To resolve this, I use a private helper function to parse the provided URI and construct the appropriate format that will let us download the repository. You know what they say: "if you have a problem and regex is the answer, you now have two problems." I'm admittedly not the best at regex, but luckily [regex101.com](https://regex101.com/) exists and I ended up with this:

```PowerShell
function constructGitHubUri($Url) {
  # Desired format: https://github.com/username/repositoryname/archive/refs/heads/main.zip
  if ($Uri -match "^https://github\.com/([^/]+)/([^/]+)(?:/)?$") {
    $user = $matches[1]
    $repo = $matches[2] -replace "\.git$", ""
    return "https://github.com/$user/$repo/archive/refs/heads/main.zip"
  }
  else {
    exit 1
  }
}
```

### GitHub or GitLab?
I originally wrote this function specifically for GitHub, but realized it could be handy to make it support GitLab as well. Unfortunately, the URI format for GitHub and GitLab are different, so I have a similar private helper function for this one, too:
```PowerShell
function constructGitLabUri($Url) {
  # Desired format: https://gitlab.com/workspacename/projectname/-/archive/main/projectname.zip
  if ($Uri -match "^https://gitlab\.com/([^/]+)/([^/]+)(?:/)?$") {
    $workspace = $matches[1]
    $repo = $matches[2] -replace "\.git$", ""
    return "https://gitlab.com/$workspace/$repo/-/archive/main/$repo-main.zip"
  }
  else {
    exit 1
  }
}
```

To determine which helper function to use, I use a `switch` statement:
```PowerShell
switch ($Uri.Split('/')[2]) {
    "github.com" {
        $downloadUrl = constructGitHubUri -url $Uri
    }
    "gitlab.com" {
        $downloadUrl = constructGitLabUri -url $Uri
    }
    default {
      exit 1
    }
}
```

### Downloading the Repository
I then make a temporary directory to store the downloaded repository, verify that the target output directory exists, download the repository as a .zip archive, and extract it to the output directory.

The name of the staged .zip archive has a random GUID included to avoid any name collisions in the event that the same filename is already present. Additonally, the script checks to validate that the output directory exists and creates it if it does not.
```PowerShell
  $tempZipPath = "$env:TEMP\repo_$(New-Guid).zip"

  if (-not (Test-Path $OutputDir)) {
      New-Item -ItemType Directory -Path $OutputDir | Out-Null
  }

  Invoke-WebRequest -Uri $downloadUrl -OutFile $tempZipPath -ErrorAction Stop

  Expand-Archive -Path $tempZipPath -DestinationPath $OutputDir -Force -ErrorAction Stop

  Remove-Item $tempZipPath -Force
```
The download and extraction is accomplished using simple, Windows-native functionality, such as `Invoke-WebRequest` for downloading the .zip archive and `Expand-Archive` for extracting it to the output directory. It was important to me that this function have no external dependencies (like needing Git installed on the machine).

### Importing the Module
At this point, it's import time! As you'd expect, I'm just using `Import-Module` as usual, but targeting the `.psm1` file in the downloaded and extracted repository:
```PowerShell
Import-Module -Name (Get-ChildItem -Path $OutputDir -Recurse -Filter *.psm1 | Select-Object -First 1).FullName
```
Using `Get-ChildItem` with the `-Recurse` and `-Filter` parameters helps in easily finding the PowerShell module file that contains the functions to import. Because of this, the function does not yet support repositories with more complex structures or multiple modules. It grabs the first `.psm1` file it finds--assuming that there is only one--which works for simple modules but might not work as expected if you in a nested or poly-module setup.

Of course, I also added some debug/verbose output and some light error handling as well but I figured I wouldn't make this post longer than needed with that.

## Considerations and Future Improvements
There are a few important caveats to know. 

First, this function assumes that the main branch is named `main`, which is the current Git default but not guaranteed. If the repository uses `master` or a custom branch name, this will not work; I intend to add a `-Branch` parameter to resolve this in the near future. 

**Security Note**  
Another thing to keep in mind is that the function doesn’t verify anything about the contents of the repo. There’s no signing check, no validation—so this is strictly a trust-based approach. You’ll want to be cautious and know what you're pulling in.

**Compliance Note**  
It is possible to use this function to get around restrictions in your environment that prevent you from installing modules directly from the PowerShell Gallery or other sources. You should comply with all regulations, company policies, licensing agreements, and other requirements/restrictions. Use responsibly and at your own risk.


## Wrapping Up
Mostly, this was just a fun little exercise in PowerShell for me. But it could potentially be useful at some point, so I'm sharing it. If you find yourself in a similar situation, I hope this helps.

The full function is available as a [GitHub Gist](https://gist.github.com/griffeth-barker/30d781904c9a9579ce360303c4ed9e93) as well as shown below:

```PowerShell
function Import-ModuleFromGitx {
    <#
    .SYNOPSIS
        Imports a PowerShell module from a GitHub repository.

    .DESCRIPTION
        This function lets you import a PowerShell module directly from a GitHub repository without needing to install the module.
        The module is downloaded from the specified GitHub repository URI, extracted, and imported into your current PowerShell session.

    .PARAMETER Uri
        The URI of the GitHub repository. This can be obtained by clicking the green "Code" button on the repository page and copying the HTTPS URL.

        Type                : String
        Required            : True
        ValueFromPipeline   : True

    .PARAMETER OutputDir
        The directory where the module will be downloaded and imported.

        Type                : String
        Required            : False
        Default Value       : $pwd

    .EXAMPLE
        # Import a PowerShell module from a GitHub or GitLab repository
        Import-ModuleFromGitx -Uri "https://github.com/username/repository.git"
        Import-ModuleFromGitx -Uri "https://gitlab.com/workspace/repository.git"

    .EXAMPLE
        # Import a PowerShell module from a GitHub or GitLab repository and specify an output directory other than the present working directory
        Import-ModuleFromGitx -Uri "https://github.com/username/repository.git" -OutputDir "$env:USERPROFILE\Downloads"
        Import-ModuleFromGitx -Uri "https://gitlab.com/workspace/repository.git" -OutputDir "$env:USERPROFILE\Downloads"

    .NOTES
        It is possible to use this function to get around restrictions in your environment that prevent you from installing modules directly from the PowerShell Gallery or other sources.
        You should comply with all regulations, company policies, licensing agreements, and other requirements/restrictions. Use at your own risk.

        Does NOT support privately hosted versions of GitLab.

    .LINK
        https://gist.github.com/griffeth-barker/30d781904c9a9579ce360303c4ed9e93
    #>

    [CmdletBinding()]
    param (
        [Parameter(Mandatory = $true, ValueFromPipeline = $true)]
        [string]$Uri,

        [Parameter(Mandatory = $false)]
        [string]$OutputDir = "$pwd"
    )

    begin {
        function constructGitHubUri($Url) {
            if ($Uri -match "^https://github\.com/([^/]+)/([^/]+)(?:/)?$") {
                $user = $matches[1]
                $repo = $matches[2] -replace "\.git$", ""
                return "https://github.com/$user/$repo/archive/refs/heads/main.zip"
            }
            else {
                Write-Error "Invalid GitHub repository URL format. Should be 'https://github.com/username/repository.git' where username and repository are the GitHub valid user and repository names."
                exit 1
            }
        }

        function constructGitLabUri($Url) {
            if ($Uri -match "^https://gitlab\.com/([^/]+)/([^/]+)(?:/)?$") {
                $workspace = $matches[1]
                $repo = $matches[2] -replace "\.git$", ""
                return "https://gitlab.com/$workspace/$repo/-/archive/main/$repo-main.zip"
                #https://gitlab.com/griffeth-barker/testing/-/archive/main/testing-main.zip
            }
            else {
                Write-Error "Invalid GitLab repository URL format. Should be 'https://gitlab.com/workspace/repository.git' where username and repository are the GitLab valid user and repository names."
                exit 1
            }
        }
    }

    process {

        switch ($Uri.Split('/')[2]) {
            "github.com" {
                Write-Debug "Detected GitHub repository; using constructGitHubUri function"
                $downloadUrl = constructGitHubUri -url $Uri
            }
            "gitlab.com" {
                Write-Debug "Detected GitLab repository; using constructGitLabUri function"
                $downloadUrl = constructGitLabUri -url $Uri
            }
            default {
                Write-Error "Unsupported repository type. Only GitHub and GitLab repositories are supported at this time."
                exit 1
            }
        }



        try {
            Write-Verbose "Creating temporary directory for download..."
            Write-Debug "Creating '$env:TEMP\repo_$(New-Guid).zip'"
            $tempZipPath = "$env:TEMP\repo_$(New-Guid).zip"

            if (-not (Test-Path $OutputDir)) {
                New-Item -ItemType Directory -Path $OutputDir | Out-Null
            }

            Write-Verbose "Downloading repository..."
            Write-Debug "Using URI: $Uri"
            Invoke-WebRequest -Uri $downloadUrl -OutFile $tempZipPath -ErrorAction Stop
            Write-Debug "File saved to $tempZipPath"

            Write-Debug "Extracting to $OutputDir"
            Expand-Archive -Path $tempZipPath -DestinationPath $OutputDir -Force -ErrorAction Stop

            Write-Debug "Removing $tempZipPath"
            Remove-Item $tempZipPath -Force

            Write-Verbose "Importing module..."
            Write-Debug "Importing module from $OutputDir"
            Import-Module -Name (Get-ChildItem -Path $OutputDir -Recurse -Filter *.psm1 | Select-Object -First 1).FullName

        }
        catch {
            Write-Error $_.Exception.Message
        }

    }

    end {

    }

}
```