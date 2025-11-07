---
title: Getting Download Links from Webpages using PowerShell
tags:
  - powershell
  - web-scraping
---
# Introduction
I recently had the need to update the Microsoft ODBC Driver for SQL Server on a collection of Windows servers. I found the download URL for the particular version I needed on Microsoft's website and then threw together a quick script that I could run against the list of servers to download and update the ODBC driver on all the servers.

Later in the day I had the thought, "that's nice to patch to a specific version, but what about just going to the latest version?" If I could devise a method to get the link for the latest version, the script wouldn't need to be updated if it needed to be used again. Who doesn't like making less work for themselves down the road?

Let's take a look at one of several ways you can get download links from webpages using PowerShell, and how I decided to use that in the improved script.
# What's in the Box?
The first step was to determine the URL of the site whose content you want to get, and get it. In this example, I'll be working with this URL:

https://learn.microsoft.com/en-us/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver16

Getting all of the website content was pretty straightforward; we can accomplish this with the `Invoke-WebRequest` command:

```PowerShell
$webpage = Invoke-WebRequest -Uri 'https://learn.microsoft.com/en-us/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver16'
```

You'll see the content returned as a big WebResponseObject that is a bit hard to read initially:

```output

StatusCode        : 200
StatusDescription : OK
Content           : <!DOCTYPE html>
                    
                    <html class="hasSidebar hasPageActions hasBreadcrumb conceptual has-defauΓÇª
RawContent        : HTTP/1.1 200 OK
                    ETag: "l2tFcmlGo/WBcYmfmCDq/h6fY1MXeSTkBGyQSV4zBoE="
                    Request-Context: appId=cid-v1:b1c5b6ea-7ff0-41d3-9862-84c5e1dc3be7
                    X-Datacenter: wus
                    X-Frame-Options: SAMEORIGIN
                    X-Content-TypΓÇª
Headers           : {[ETag, System.String[]], [Request-Context, System.String[]], [X-Datacenter, System.String[]], [X-Frame-Options, System.String[]]ΓÇª}
Images            : {@{outerHTML=<img src="../../includes/media/yes-icon.svg?view=sql-server-ver16" role="presentation" data-linktype="relative-path">; tagName=IMG; src=../../includes/media/yes-icon.svg?view=sql-server-ver16; role=presentation; data-linktype=relative-path}, @{outerHTML=<img src="../../includes/media/yes-icon.svg?view=sql-server-ver16" role="presentation" data-linktype="relative-path">; tagName=IMG; src=../../includes/media/yes-icon.svg?view=sql-server-ver16; role=presentation; data-linktype=relative-path}, @{outerHTML=<img src="../../includes/media/yes-icon.svg?view=sql-server-ver16" role="presentation" data-linktype="relative-path">; tagName=IMG; src=../../includes/media/yes-icon.svg?view=sql-server-ver16; role=presentation; data-linktype=relative-path}, @{outerHTML=<img src="../../includes/media/yes-icon.svg?view=sql-server-ver16" role="presentation" data-linktype="relative-path">; tagName=IMG; src=../../includes/media/yes-icon.svg?view=sql-server-ver16; role=presentation; data-linktype=relative-path}ΓÇª}
InputFields       : {}
Links             : {@{outerHTML=<a href="#main" class="skip-to-main-link has-outline-color-text visually-hidden-until-focused position-fixed has-inner-focus focus-visible top-0 left-0 right-0 padding-xs has-text-centered has-body-background" tabindex="1">Skip to main content</a>; tagName=A; href=#main; class=skip-to-main-link has-outline-color-text visually-hidden-until-focused position-fixed has-inner-focus focus-visible top-0 left-0 right-0 padding-xs has-text-centered has-body-background; tabindex=1}, @{outerHTML=<a href="https://go.microsoft.com/fwlink/p/?LinkID=2092881 "
                    						style="
                    						background-color: #0078d4;
                    						border: 1px solid #0078d4;
                    						color: white;
                    						padding: 6px 12px;
                    						border-radius: 2px;
                    						display: inline-block;
                    						">
                    Download Microsoft Edge					</a>; tagName=A; href=https://go.microsoft.com/fwlink/p/?LinkID=2092881 ; style=
                    						background-color: #0078d4;
                    						border: 1px solid #0078d4;
                    						color: white;
                    						padding: 6px 12px;
                    						border-radius: 2px;
                    						display: inline-block;
                    						}, @{outerHTML=<a href="https://learn.microsoft.com/en-us/lifecycle/faq/internet-explorer-microsoft-edge"
                    						style="
                    							background-color: white;
                    							padding: 6px 12px;
                    							border: 1px solid #505050;
                    							color: #171717;
                    							border-radius: 2px;
                    							display: inline-block;
                    							">
                    More info about Internet Explorer and Microsoft Edge					</a>; tagName=A; href=https://learn.microsoft.com/en-us/lifecycle/faq/internet-explorer-microsoft-edge; style=
                    							background-color: white;
                    							padding: 6px 12px;
                    							border: 1px solid #505050;
                    							color: #171717;
                    							border-radius: 2px;
                    							display: inline-block;
                    							}, @{outerHTML=<a itemprop="url" href="https://www.microsoft.com" aria-label="Microsoft" class="nav-bar-button">
                    				<div class="nav-bar-logo has-background-image theme-display is-light" role="presentation" aria-hidden="true" itemprop="logo" itemscope="itemscope"></div>
                    				<div class="nav-bar-logo has-background-image theme-display is-dark is-high-contrast" role="presentation" aria-hidden="true" itemprop="logo" itemscope="itemscope"></div>
                    			</a>; tagName=A; itemprop=url; href=https://www.microsoft.com; aria-label=Microsoft; class=nav-bar-button}ΓÇª}
RawContentLength  : 65147
RelationLink      : {}



```

# Just the Links, Please
What we really want to focus on here is the Links property of the object (I've truncated it for the sake of brevity in the post, since it's quite lengthy):

```PowerShell
$webpage.links
```

```output

outerHTML : <a href="#main" class="skip-to-main-link has-outline-color-text visually-hidden-until-focused position-fixed has-inner-focus focus-visible top-0 left-0 right-0 padding-xs has-text-centered has-body-background" tabindex="1">Skip to main content</a>
tagName   : A
href      : #main
class     : skip-to-main-link has-outline-color-text visually-hidden-until-focused position-fixed has-inner-focus focus-visible top-0 left-0 right-0 padding-xs has-text-centered has-body-background
tabindex  : 1

outerHTML : <a href="https://go.microsoft.com/fwlink/p/?LinkID=2092881 "
            						style="
            						background-color: #0078d4;
            						border: 1px solid #0078d4;
            						color: white;
            						padding: 6px 12px;
            						border-radius: 2px;
            						display: inline-block;
            						">
            Download Microsoft Edge					</a>
tagName   : A
href      : https://go.microsoft.com/fwlink/p/?LinkID=2092881 
style     : 
            						background-color: #0078d4;
            						border: 1px solid #0078d4;
            						color: white;
            						padding: 6px 12px;
            						border-radius: 2px;
            						display: inline-block;
            						

...
```

# This is the Link You're Looking For
This is better, since it is only returning the links for the page content, however we need to take it one step further. Looking at the webpage, I know that the displayed text for the download link is "Download Microsoft ODBC Driver 17 for SQL Server (x64)". What I ended up doing was moving deeper into the `Invoke-WebRequest`'s response, specifically into the `OuterHTML`. From here, we can use `Select-String` to find the specific URL that we want:

```PowerShell
$webpage.links.outerhtml | Select-String -SimpleMatch "Download Microsoft ODBC Driver 17 for SQL Server (x64)" | Out-String
```

This left me with a this line of the HTML, as a string:

```output
<a href="https://go.microsoft.com/fwlink/?linkid=2249004" data-linktype="external">Download Microsoft ODBC Driver 17 for SQL Server (x64)</a>
```

We can then use basic string manipulation to get our our target URL. Given that the URL is wrapped in double quotation marks, we can split the string at the double quotation marks and select one of the splits:

```PowerShell
$url = ($webpage.links.outerhtml | Select-String -SimpleMatch "Download Microsoft ODBC Driver 17 for SQL Server (x64)" | Out-String).split('"')[1]
```

The result is our desired download URL:

```output
https://go.microsoft.com/fwlink/?linkid=2249004
```

# Conclusion
With my download URL obtained, I was able to write an improved version of the script which will install the *latest* update for the Microsoft ODBC Driver for SQL Server, rather than a specified version.

Is there a better way to accomplish this? Quite possibly. As the saying goes, there's more than one way to cook an egg. This was just thrown together out of necessity. 

If you're interested, you can check out how I ended up implementing this in the script [in my public GitHub repository](https://github.com/griffeth-barker/public/blob/main/powershell/update_msodbcsql_driver_latest.ps1). I hope that you've found something in this post helpful in some way!