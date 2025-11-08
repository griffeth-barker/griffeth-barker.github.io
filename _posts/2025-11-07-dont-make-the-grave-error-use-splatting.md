---
title: "Don't Make the Grave Error: Use Splatting"
date: "2025-11-07"
tags:
  - powershell
  - formatting
categories:
  - powershell
---

# Introduction
Have you ever been in the situation where you needed to pass a bunch of parameters to a function, but they started to go off the edge of the screen in your ISE/IDE? You probably have. Maybe you used the grave ( \` ) character to safely extend the command over multiple lines.


```powershell
# Long one-liner
New-DistributionGroup -Name 'MyDistributionGroup' -Description 'My distribution group' -PrimarySmtpAddress 'MyDistributionGroup@domain.tld' -Alias 'MyDistributionGroup' Members 'Person1@domain.tld', 'Person2@domain.tld'
```

```powershell
# Long one-liner split across multiple lines using graves
New-DistributionGroup `
    -Name 'MyDistributionGroup' `
    -Description 'My distribution group' `
    -PrimarySmtpAddress 'MyDistributionGroup@domain.tld' `
    -Alias 'MyDistributionGroup' `
    -Members 'Person1@domain.tld', 'Person2@domain.tld'
```

Now, that probably seems better initially, but we can do better and avoid having to remember to use the grave symbol at the end of each line.

# Splatting
Splatting is a method of passing a hashtable of parameters to a function or command as a single unit, making code more readable and easier to maintain.
Let's take the above example and use splatting instead:

```powershell
$distGroupParams = @{
    Name               = 'MyDistributionGroup'
    Description        = 'My distribution group'
    PrimarySmtpAddress = 'MyDistributionGroup@domain.tld'
    Alias              = 'MyDistributionGroup'
    Members            = 'Person1@domain.tld', 'Person2@domain.tld'
    WhatIf             = $true
}
New-DistributionGroup @distGroupParams
```

Note that when we use splatting, we call the hashtable using `@` instead of `$`.

# Benefits
There are a few benefits to using splatting. First, it makes the code easier to read and understand. Second, it reduces the chance of making a mistake and forgetting to add a grave symbol at the end of each line. Another added benefit is that the parameters are now easily modifiable programatically if needed. For example, we could change WhatIf from `$true` to `$false` if we wanted to:

```powershell
$distGroupParams.WhatIf = $false
```

Not only can that be done in a one-off, but it can also be done as part of a switch or if/else statement.

# Wrapping Up
So what do you think? Is splatting better? Your answer should be "yes!" Not only does it offer more readable and maintainable code, but it also is considered the generally accepted best practice when writing PowerShell scripts. If you're interested in learning more about splatting, I recommend checking out [about_Splatting - PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_splatting?view=powershell-7.3).