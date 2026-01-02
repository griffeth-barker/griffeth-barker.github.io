---
title: "Moving Beyond PSCustomObjects with PowerShell Classes"
tags:
  - powershell
  - classes
  - objects
  - object-oriented-programming
categories:
  - powershell
---

We've all probably been there. You're working with some platform's API and it returns you a veritable "bag" of properties in a JSON string or something similar. 

Sometimes some properties are strings, but sometimes some of them are arrays of strings. 

Sometimes you're re-running the same line of code again to get an updated copy of the data. 

If you want to do anything else with that information, you probably have to pass it off to external functions. There are all kinds of little irritations, especially with some platform's APIs. 

Most newer PowerShell folks will have discovered casting to a PSCustomObject, but that's not the best action we can take. So let's move beyond them with classes.

## Introduction
Think of classes as blueprints used to create structured objects. We can create our own kind of "PSCustomObject" with features more relevant to the situation with which we're working. In this blog post, we'll be working with a handy website [ifconfig.me](https://ifconfig.me). This website provides you information about your computer's identity as seen by a web server out on the internet. This is handy for checking your public IP address, among other things. If you browse to the site, you'll see an HTML page rendered with a table of information. But we can interact with this site using `Invoke-RestMethod` in PowerShell:

```powershell
Invoke-RestMethod -Method GET -Uri "https://ifconfig.me/all.json"

# Output:
# ip_addr    : xxx.xxx.xxx.xxx
# user_agent : Mozilla/5.0 (Linux; Garuda Linux; en-US) PowerShell/7.5.4
# port       : 52562
# method     : GET
# encoding   : gzip, deflate, br
# via        : 1.1 google
# forwarded  : xxx.xxx.xxx.xxx,yyy.yyy.yyy.yyy
```

Alright, so this output, which is automatically made a PSCustomObject, really isn't that bad, but it's simple to work with for demonstrative purposes. Let's see how we can improve this!

## Class Structure
The essential structure of a PowerShell class looks like this:

```powershell
class ClassName {
  # visible properties
  [type]$property

  # hidden properties
  hidden [type]$property

  # constructor
  ClassName([type]$apiResponse) {
    $this.Property = $apiResponse.rawProperty
  }

  # instance methods
  [type] MethodName() {
    # do something with an input object
    # return something
  }

  # static methods
  static [type] MethodName() {
    # do something without an input object
    # return something
  }
}
```

So let's build out a `WebIdentityInfo` class for the data we get back from [ifconfig.me](https://ifconfig.me).

## Properties
We are getting back:  
  - ip_addr
  - user_agent
  - port
  - method
  - encoding
  - via
  - forwarded

I don't particularly like having underbars in property names, so I'd want to fix that, along with using proper casing for the property names. Right now, each property's value is just a string. It would be nice for the port to be an integer and the encoding and forwarded properties to be arrays, since they often have more than one item in the string.

Let's add the properties we want the `WebIdentityInfo` class to have:

```powershell
class WebIdentityInfo {
    [System.String]$IPAddress
    [System.String]$UserAgent
    [System.Int32]$TcpPort
    [System.String][ValidateSet("GET", "PATCH", "POST", "PUT", "DELETE")]$Method
    [System.Array]$Encoding
    [System.String]$Via
    [System.Array]$Forwarded
}
```

At this point, we've removed the underbars from the property names and fixed the casing issues. You'll also notice that like in regular scripts and functions, you can decorate these parameters as well. I've done so on one parameter for illustrative purposes. 

## Constructor
With the parameters defined, let's lay out how the object will actually be constructed. We do this with a constructor (imagine that).

```powershell
class WebIdentityInfo {
    # ... properties ...
    $this.IPAddress     = $apiResponse.ip_addr
    $this.UserAgent     = $apiResponse.user_agent
    $this.TcpPort       = $apiResponse.port
    $this.Method        = $apiResponse.method
    $this.Via           = $apiResponse.via
    # here will be Encoding and Forwarded
}
```

> Note that `$apiResponse` really could be called just about anything other than automatic variable names.

While we were already We still need to add lines for Encoding and Forwarded, so let's add some logic for that:

```powershell
    # ... properties ...
    # ... other constructor components ...
    if ( $apiResponse.encoding ) {
        $this.Encoding  = $apiResponse.encoding.Split(',').Trim()
    } else {
        $this.Encoding = @()
    }
    if ( $apiResponse.forwarded ) {
        $this.Forwarded = $apiResponse.forwarded.Split(',').Trim()
    } else {
        $this.Forwarded = @()
    }
```

With this logic, regardless of whether the comma-separated strings contain 0, 1, or more items, it will be handled. This is also nice as it will enable indexing into the arrays without having to manually parse the string values and do `.Split(',').Trim()` manually each time.

Our finished constructor looks like this:

```powershell
    # ... properties ...
    WebIdentityInfo([System.Object]$ApiResponse) {
        $this.IPAddress     = $ApiResponse.ip_addr
        $this.UserAgent     = $ApiResponse.user_agent
        $this.TcpPort       = $ApiResponse.port
        $this.Method        = $ApiResponse.method
        $this.Via           = $ApiResponse.via
        if ( $ApiResponse.encoding ) {
            $this.Encoding  = $ApiResponse.encoding.Split(',').Trim()
        } else {
            $this.Encoding = @()
        }
        if ( $ApiResponse.forwarded ) {
            $this.Forwarded = $ApiResponse.forwarded.Split(',').Trim()
        } else {
            $this.Forwarded = @()
        }
    }
```

We're expecting any `System.Object` as the input object type and property mapping will happen as we designed.

## Methods
Now let's go a bit beyond the basics. We're going to introduce Object-Oriented Programming concepts here. You may have noticed when using PowerShell that if you pipe certain objects to `Get-Member` that there are not only properties of the object, but sometimes *methods* as well. In fact, we used some methods earlier when we used `.Split(',')` and `.Trim()`. There are two types of methods we can add--instance and static. Instance methods require an instance of the object to already exist (e.g. `$object.Method()`), whereas static methods do not require an instance of the object to exist, and often create an instance of the object (e.g. `[ClassName]::Method()`).

Here's our list of things we want to be able to do via methods:
  - Try to ping our public IP address as returned by the API call
  - Try to connect to our public IP address on the TCP port returned by the API call
  - Output our web identity info to a CSV file
  - Output our web identity info as JSON text
  - Refresh our web identity info if it was stored as a variable
  - Get WHOIS information for our public IP address
  - Create the object

### Instance Methods
#### TestICMP()
We basically want to write this like a function, name it as a method, and use `Test-Connection` here. We will use a `try`/`catch` block to determine if the ping was successful:

```powershell
    [System.Boolean] TestICMP() {
        try {
            Test-Connection -ComputerName $this.IPAddress -Count 1 -ErrorAction Stop
            return $true
        }
        catch {
            return $false
        }
    }
```

#### TestTCP()
Similar to the ping version, we'll use `Test-Connection` in a `try`/`catch` block, but with `-TcpPort $this.TcpPort`

#### ToCSV()
Outputting the information to a CSV file is pretty easy, too. We want to let the user provide a filepath if they desire, but if not just write the file to the current directory. For best practice, we'll also return some file information as would be normal expected behavior in Powershell.

```powershell
    [System.IO.FileInfo] ToCSV([System.String]$Path) {
        if ( -not $Path ) {
            $Path = Join-Path -Path $PWD -ChildPath "WebIdentityInfo.csv"
        }

        try {
            $this | Export-Csv -Path $Path -NoTypeInformation -Force
            return Get-Item -Path $Path
        }
        catch {
            throw "Failed to export WebIdentityInfo to CSV file at path '$Path': 
            $($_.Exception.Message)"
        }
    }
```

The `catch` block looks a bit different this time. Since we are not just returning one of the two boolean values, we want to include an error message.

#### ToJSON()
Outputting as JSON is extremely straightforward, and exactly as you'd expect.

```powershell
    [System.String] ToJSON() {
        return $this | ConvertTo-Json -Depth 3
    }
```

#### Refresh()
Now this one is a bit different. If the user has saved the object as a variable, they may want to be able to refresh the data without creating a new instance of the object. To do this, we will call the API again and update the properties of the object.

This will be a bit "cart before the horse", as we've yet to define the actual code that will create an instance of the object, but we will get there shortly.

```powershell
    [void] Refresh() {
        try {
            $refreshQueryParams = @{
                Method = 'GET'
                Uri    = 'https://ifconfig.me/all.json'
                ErrorAction = 'Stop'
            }
            $newQuery = Invoke-RestMethod @refreshQueryParams

            $this.IPAddress = $newQuery.ip_addr
            $this.UserAgent = $newQuery.user_agent
            $this.TcpPort   = $newQuery.port
            $this.Method    = $newQuery.method
            $this.Via       = $newQuery.via
            if ( $newQuery.encoding ) {
                $this.Encoding  = $newQuery.encoding.Split(',').Trim()
            } else {
                $this.Encoding = @()
            }
            if ( $newQuery.forwarded ) {
                $this.Forwarded = $newQuery.forwarded.Split(',').Trim()
            } else {
                $this.Forwarded = @()
            }
            $this.Timestamp = [System.DateTime]::UtcNow
        }
        catch {
            throw "Failed to refresh network identity information: $($_.Exception.Message)"
        }
    }
```

Earlier in this post, remember how we defined a hidden Timestamp property? We update that when using the `Refresh()` method.

#### GetWhoIs()
Let's also offer a method to get some information about the owner of the public IP address returned by the API call. We can just use the ARIN WHOIS Rest API for this.

> There are certainly things to be said about external dependencies in methods. This is purely for illustrative purposes and does not denote best practice.

```powershell
    [System.Object] GetWhois () {
        try {
            $queryParams = @{
                Method = 'GET'
                Uri    = "https://whois.arin.net/rest/ip/$($this.IPAddress)"
                ErrorAction = 'Stop'
            }
            $query = (Invoke-RestMethod @queryParams).net # the data we want is nested one level down
            return $query
        }
        catch {
            throw "Failed to retrieve WHOIS information for IP address $($this.IPAddress): 
            $($_.Exception.Message)"
        }
    }
```

### Static Methods
Finally and crucially, we need a static method that allows us to create an instance of the `WebIdentityInfo` object class.

```powershell
    static [WebIdentityInfo] Get() {
        try {
            $query = Invoke-RestMethod -Method GET -Uri 'https://ifconfig.me/all.json'
            return [WebIdentityInfo]::new($query)
        }
        catch {
            throw "Failed to retrieve network identity information: $($_.Exception.Message)"
        }
    }
```
> Naming things is hard.

## Usage
**You can find the completed class definition in this [GitHub gist](https://gist.github.com/griffeth-barker/974a573e850a6e9ce197ba1d776e32a6).**

```powershell
# Create an object
$info = [WebIdentityInfo]::Get()
# No output

# View properties and members
$info | Get-Member
# Output:
#    TypeName: WebIdentityInfo

# Name        MemberType Definition
# ----        ---------- ----------
# Equals      Method     bool Equals(System.Object obj)
# GetHashCode Method     int GetHashCode()
# GetType     Method     type GetType()
# GetWhois    Method     System.Object GetWhois()
# Refresh     Method     void Refresh()
# TestICMP    Method     bool TestICMP()
# TestTCP     Method     bool TestTCP()
# ToCSV       Method     System.IO.FileInfo ToCSV(string Path)
# ToJSON      Method     string ToJSON()
# ToString    Method     string ToString()
# Encoding    Property   array Encoding {get;set;}
# Forwarded   Property   array Forwarded {get;set;}
# IPAddress   Property   string IPAddress {get;set;}
# Method      Property   string Method {get;set;}
# TcpPort     Property   int TcpPort {get;set;}
# UserAgent   Property   string UserAgent {get;set;}
# Via         Property   string Via {get;set;}

# Ping the public IP address
$info.TestICMP()
# Output
# True

# Ping the public IP address on the TCP port
$info.TestTCP()
# Output
# True

# Write info to CSV file
$info.ToCSV('~/Desktop/WebIdentityInfo.csv')
# Output
#     Directory: /home/griff/Desktop

# UnixMode         User Group         LastWriteTime         Size Name
# --------         ---- -----         -------------         ---- ----
# -rw-r--r--      griff griff        1/1/2026 23:21          213 WebIdentityInfo.csv

# Output as JSON
$info.ToJSON()
# Output
# {
#   "IPAddress": "xxx.xxx.xxx.xxx",
#   "UserAgent": "Mozilla/5.0 (Linux; Garuda Linux; en-US) PowerShell/7.5.4",
#   "TcpPort": 52780,
#   "Method": "GET",
#   "Encoding": [
#     "gzip",
#     "deflate",
#     "br"
#   ],
#   "Via": "1.1 google",
#   "Forwarded": [
#     "xxx.xxx.xxx.xxx",
#     "yyy.yyy.yyy.yyy"
#   ],
#   "Timestamp": "2026-01-02T07:18:35.6878374Z",
#   "InstanceId": "74a066c5-2328-4bde-b02c-f67fc6500ea6"
# }

# Refresh the object's data
$info.Refresh()
# Note the InstanceID and Timestamp in the JSON example. The InstanceID of $info would
# remain the same but any new data along with Timestamp would be updated.

# Get WHOIS info
$info.GetWhoIs()
# Output
# xmlns               : https://www.arin.net/whoisrws/core/v1
# ns2                 : https://www.arin.net/whoisrws/rdns/v1
# ns3                 : https://www.arin.net/whoisrws/netref/v2
# copyrightNotice     : Copyright 1997-2026, American Registry for Internet Numbers, Ltd.
# inaccuracyReportUrl : https://www.arin.net/resources/registry/whois/inaccuracy_reporting/
# termsOfUse          : https://www.arin.net/resources/registry/whois/tou/
# registrationDate    : 2023-12-13T15:43:46-05:00
# rdapRef             : https://rdap.arin.net/registry/ip/216.200.120.96
# ref                 : https://whois.arin.net/rest/net/NET-216-200-120-96-1
# customerRef         : customerRef
# endAddress          : xxx.xxx.xxx.xxx
# handle              : NET-xxx
# name                : ZAYO-xxx
# netBlocks           : netBlocks
# resources           : resources
# parentNetRef        : parentNetRef
# comment             : comment
# startAddress        : xxx.xxx.xxx.xxx
# updateDate          : 2023-12-13T15:43:46-05:00
# version             : 4
```

## Final Thoughts
Transitioning from simple scripts that work to reliable systems requires changing how we handle data. Moving away from generic PSCustomObjects to classes lets us create better formatted data that is type-safe, more pipe-able, and consistent. Things like data normalization can happen once when the data is obtained, rather than manually in the script or at the command line each time. We also unlock a whole new world of shortcuts with methods. And if the underlying API ever changes, we need only update the class definition, rather than every function and script file that uses that API.

I went a long time not using classes in modules and other PowerShell work I did because I struggled to understand them. I hope that this post walks through the basic process of writing a class in a way that helps someone along their PowerShell journey. 

Have you used classes before? Let me know [@griff.systems on BlueSky](https://bsky.app/profile/griff.systems).

## Additional Reading
  - [Microsoft Learn - PowerShell - about_Classes](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_classes?view=powershell-7.5)
