---
title: Fixing PowerShell Remote Console in Devolutions Remote Desktop Manager
tags:
  - break-fix
  - remote-desktop-manager
  - devolutions
  - powershell
  - terminal
---
> **Special Thanks**
> *Special thanks to [**Marc-André Moreau**](https://forum.devolutions.net/users/25115/mamoreau), whose expertise and assistance on the Devolutions Forum led to this workaround.*

> **Disclaimer**
> *Please note that I am not affiliated with nor supported by Devolutions or their staff, and if you experience issues with your Remote Desktop Manager application, you should probably contact their technical support or post on their forum. I am not an RDM developer, thus the explanations contained herein are the result of my best understanding based on interactions with Devolutions staff, and are not guaranteed to be completely accurate. As always, you should consult technical support for any issues; additionally, you should fully understand fully any command or script before running them on your or any other system.*

# Problem
Launching a Remote PowerShell Console session entry in Devolutions Remote Desktop Manager results in the following error:

```output
Invoke-Expression : Exception setting "BufferSize": "Cannot set the buffer size because the size specified is too large or too small.
Parameter name: value
Actual value was 192,4000."
At line:32 char:13
+             Invoke-Expression $SecureCommand
+             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Invoke-Expression], SetValueInvocationException
    + FullyQualifiedErrorId : ExceptionWhenSetting,Microsoft.PowerShell.Commands.InvokeExpressionCommand
```

# Environment
I identified this issue in Remote Desktop Manager version 2023.2.28.0, though Devolutions forums indicate such issues may have existed in prior versions as well. The workstation with RDM installed was Windows 11 23H2 with Windows PowerShell 5.1 as the default PowerShell environment, though the issue appears to be operating system agnostic, as well as architecture agnostic.

During troubleshooting, I found that if I installed PowerShell 7 and set the entry's terminal type to PowerShell 7 instead of Default or Windows PowerShell 5.1 and changed the shell type from Console Host to Windows Terminal, that I was able to use the session entry as expected, though this requires the deployment of PowerShell 7 to RDM users, which may not meet the needs of all organizations and environments.

# Background
RDM's PowerShell Remote Console session type is supposed to natively handle both Windows PowerShell 5.1 and PowerShell 7.

When RDM opens the PowerShell Remote Console session, RDM calls `conhost.exe` then resizes it using an injected command and reparents it into a docked tab within the RDM application interface. The specified PowerShell environment appears to then be opened inside of the Console Host, and then an inbuilt script is run which establishes a PSSession to the host specified in the session entry's properties.

The Console Host was not intended to be embedded into other applications. Because of this, the Console Host must be resized before reparenting it into RDM. 

> **Note on Windows Terminal**
> Windows Terminal, the new terminal made default in Windows 11, does not support reparenting at all; if you do choose the Windows Terminal option in the session entry's properties, RDM will use a [Devolutions custom distribution of the Microsoft Windows Terminal](https://github.com/Devolutions/wt-distro), which was developed specifically to support integration with RDM via launching with a parent window, avoiding the need for reparenting after it has launched.

# Cause
This issue results from this resizing and reparenting process. When the Console Host is resized, there are two minimums which need to be taken into account:
- `$Host.UI.RawUI.BufferSize`
- `$Host.UI.RawUI.WindowSize`

The Console Host's window size must be changed before the buffer size is set. The code to shrink the window *first* is not yet existent in RDM. If the Buffer size is set first and the values for this property are incompatible with the Window size, the exception shown in the Problem section of this article will be thrown.

# Workaround
The minimum Buffer Size is determined by GetSystemMetrics(SM_CXMIN) + GetSystemMetrics(SM_CYMIN) in Windows. These values can be determined using the following code snippet:

```PowerShell
Add-Type -TypeDefinition @"
    using System.Runtime.InteropServices;

    public class User32 {
        [DllImport("user32.dll")]
        public static extern int GetSystemMetrics(int nIndex);
    }
"@

$cxmin = [User32]::GetSystemMetrics(28) # SM_CXMIN
$cymin = [User32]::GetSystemMetrics(29) # SM_CYMIN
Write-Host "SM_CXMIN: $cxmin SM_CYMIN: $cymin"
```

Sample output:
```output
SM_CXMIN: 136 SM_CYMIN: 39
```

I set my PowerShell terminal's Window size and Buffer size to these values using:
```
$Host.UI.RawUI.WindowSize = New-Object System.Management.Automation.Host.Size 136,39
$Host.UI.RawUI.BufferSize = New-Object System.Management.Automation.Host.Size = 136,39
```

> Note that if you were to run these two commands in reverse order the first time, you'd likely have the same exception described in the Problem section of this post thrown again.

In Remote Desktop Manager, go to *File > Options > Types > Other* and then locate the PowerShell section. I changed the Buffer size Rows to the `136` value and the Buffer size Cols to the `39` value, though the Cols value then forced itself to 50, which I left alone.

In the same section, I also changed the Window size Rows to the `136` value and the Window size Cols value to the `39`. Save these options changes.

Then, for the session entry in question, navigate to *Properties > General > Advanced* and uncheck the **Resize window** checkbox, then save the changes.

At this point, I was be able to launch the session entry, the process described in the Background section of this post worked properly, and the end result was a PSSession to the host defined in the session entry's properties.

# Additional Resources
- This is the forum post that I revived where I discussed the issue with a developer from Devolutions and eventually reached the workaround provided in this article, despite it being a slightly different issue: [Embedded powershell tools - potential bug found with it.. (devolutions.net)](https://forum.devolutions.net/topics/31127/embedded-powershell-tools--potential-bug-found-with-it)
- You can find the Devolutions-developed distribution of Windows Terminal on [GitHub](https://github.com/Devolutions/wt-distro).
- Information on the SetConsoleScreenBufferSize function in the Console API can be found on [Microsoft Learn](https://learn.microsoft.com/en-us/windows/console/setconsolescreenbuffersize).