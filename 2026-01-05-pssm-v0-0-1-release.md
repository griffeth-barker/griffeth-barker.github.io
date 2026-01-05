---
title: "PowerShell Service Manager v0.1.1 Release"
tags:
  - powershell
  - windows-services
  - windows
  - windows-presentation-framework
  - winforms
  - executables
categories:
  - powershell
---

PowerShell Service Manager v0.1.1 is now available! PSSM is another service manager for Microsoft Windows. You can find the project on [GitHub](https://github.com/griffeth-barker/PSSM).

# Why PSSM
Windows already has `services.msc`, so why even bother writing another service manager in PowerShell?  
  
In my career, I've often had peers who were vehemently married to "ClickOps" and simply would not learn how to do anything from the command line. To each their own. In such situations, it is sometimes necessary to build tooling that enables them to better and more easily do their work, but without the shock of moving to the command line. Sometimes, knowing that such tooling was built with PowerShell can even act as a carrot on a stick, so to speak, planting a seed to move beyond the mouse.  
  
There are somethings that myself and folks I've known frequently want to know about a service, and the more quickly we see that information, the better. So on a whim the other night, PSSM was born.  
  
# Methodology
In essence, PSSM is a PowerShell script that loads a couple of assemblies, defines a bunch of functions, and then uses logic for when to call those functions; the script is then wrapped as a standalone executable using PS2EXE.  

## Assemblies
The application script uses two assemblies: `PresentationFramework` and `System.Windows.Forms`.  
`PresentationFramework` is used to handle the bulk of the graphical user interface including the main window, buttons, and modals.  
`System.Windows.Forms` is used for the out-of-the-box windows such as the file selector browser and the like.  

The main application window is defined by a block of XAML, and then read in as an object:
```powershell
    #region main xaml -------------------------------------------------------------------------------------
    [xml]$xaml = @"
<Window xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="PowerShell Service Manager" Height="650" Width="1250" 
        WindowStartupLocation="CenterScreen" Background="#2D2D2D">
    <Window.Resources>
        <Style TargetType="Button">
            <Setter Property="Background" Value="#444444"/><Setter Property="Foreground" Value="White"/><Setter Property="BorderBrush" Value="#555555"/>
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="Button">
                        <Border Name="border" BorderThickness="1" Padding="5" BorderBrush="{TemplateBinding BorderBrush}" Background="{TemplateBinding Background}"><ContentPresenter HorizontalAlignment="Center" VerticalAlignment="Center"/></Border>
                        <ControlTemplate.Triggers>
    # TRUNCATED FOR BLOG POST DUE TO XAML LENGTH
"@
    #endregion main xaml ----------------------------------------------------------------------------------

# READ IN XAML TO BUILD WINDOW OBJECT
$Window = [Windows.Markup.XamlReader]::Load((New-Object System.Xml.XmlNodeReader $xaml))
```

## Functions
If there is something of substance that happens in the application, there's probably a function behind it. Some examples include:  
  
| Function               | Purpose                                                |
| ---------------------- | ------------------------------------------------------ |
| `Get-Uptime`           | Calculates how long a service has been running by comparing current datetime to process start time of service's currently running PID. |
| `Test-IsPermissive`    | Checks the Access Control List (ACL) of the service's folder to identify if non-admin groups have Write or Modify permissions. |
| `Update-UI`            | Refreshes data via CIM instances, builds a collection, binds them to main window's view. |
| `Show-DetailsModal`    | Launches a secondary window with the same theme, displaying extended info for the selected service. |
| `Show-InstallModal`    | Launches a secondary window with the same theme, for installing a new service. |
| `Show-DeleteModal`     | Launches a secondary window with the same theme, for deleting the selected service. | 
| `Open-ServiceRegistry` | Updates the registry editor's LastKey value then launches it to jump directly to the selected service's configuration. |
  
A more complete list of these is available in the repository's development documentation.  

## Wrapping as an Executable
With each change made, I would launch the script like so:
```powershell
Start-Process -FilePath 'powershell.exe' -ArgumentList '-STA -File .\pssm.ps1'
```
Once I was somewhat happy with the minimum viable product, I used the [PS2EXE](https://github.com/MScholtes/PS2EXE) module to wrap the script as a standalone executable:
```powershell
#requires -Module PS2EXE

$params = @{
    inputFile    = '.\src\pssm.ps1'
    outputFile   = '.\releases\pssm-0.1.1.exe'
    STA          = $true
    iconFile     = '.\images\pssm-gear-icon.ico'
    noConsole    = $true
    title        = 'PowerShell Service Manager'
    description  = 'Another service manager for Microsoft Windows'
    company      = 'griff.systems'
    version      = '0.1.1'
    requireAdmin = $true
    configFile   = $false
}

Invoke-PS2EXE @params
```
I ran the executable and verified that it functioned as intended, and here we are!  

# Performance Considerations
This is not the most performant application in existence. It is also not more performant than `services.msc`. There's about a 5 second delay on launch while the main view's data is populated. I do intend to improve performance in future releases, but the point of this was not to be more performant, but meet specific needs and niceties I (and others with whom I've worked) wanted.

# Wrapping Up
While it's nothing fancy or groundbreaking (or even new for that matter), the fact that we can load assemblies in PowerShell to draw windows and interact with things like CIM will never really cease to amaze me. This is just a simple example of the "if you can think of it, you can probably do it" aspect of PowerShell.  
  
Hopefully this post has been either interesting or helpful to at least somebody! If you find the project helpful in any way, please give it a star ⭐ on GitHub. How have you used PowerShell and loading assemblies to create graphical user interfaces? I'd love to hear from you [@griff.systems on BlueSky](https://bsky.app/profile/griff.systems).
