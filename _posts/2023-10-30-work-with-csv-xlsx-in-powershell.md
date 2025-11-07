---
title: Working with CSV and XLSX Files in PowerShell
tags:
  - powershell
  - excel
  - csv
  - data
---
# Introduction
PowerShell is a powerful scripting language that can automate a wide variety of tasks. One common activity that IT professionals need to perform is working with data files such as CSV and XLSX files, both of which are typically associated with Microsoft Excel or other workbook/worksheet applications. Excel files are widely used for storing and analyzing data, while CSV files are simple text files that contain comma-separated values but can be displayed in Excel similarly to an Excel workbook, albeit without formatting.

In this blog post, we will take a look at some low-level methods of using PowerShell to read, write, and manipulate Excel and CSV files. You will learn how to use built-in cmdlets, such as `Import-Csv` and `Export-Csv`, as well as a third-party module, **ImportExcel**, to perform basic work with these data formats. By the end of this blog post, you will be able to easily use PowerShell to handle CSV and XLSX files!

# Working with Comma-separated Values Files
PowerShell natively supports interacting with CSV files, which can enable you to get datasets into your scripts, and also provides an easy way to create objects to interact with in your scripts.

## Importing CSV Files
To import a CSV file into your terminal, use can use:

```PowerShell
Import-CSV -Path 'C:\Path\To\Your\File.csv'
```

This command will read the content of the file into the terminal and create a table-like object from the data contained in the CSV file. Often times you'll want to set the above command to a variable like so:

```PowerShell
$var = Import-CSV -Path 'C:\Path\To\Your\File.csv'
```

Once this has been completed, you can run `$var` and you should see the object printed to the terminal:

```output
griff@vm923762 ~  $var = import-csv -path 'C:\temp\excel.csv'
griff@vm923762 ~  $var

index color
----- -----
1     red
2     blue
3     yellow
4     green
5     orange
```

You can then reference properties of the object using all your usual preferred methods of interacting with objects. In this example I'm simply using the dot operator to select each individual property of the object (the columns), and also using a `Where-Object` to filter down to a specific "row" of the object.

```output
griff@vm923762 ~ $var.index
1
2
3
4
5

griff@vm923762 ~ $var.color
red
blue
yellow
green
orange

griff@vm923762 ~ $var | Where-Object { $_.index -eq 1 }

index color
----- -----
1     red
```

The above shows a foundation of how this can be useful in your automations, as you can load a dataset into PowerShell and then interact with the object as you would any other object, especially iterating through the object.

## Modifying CSV Files
We can modify the data in the object which resulted from our import of the CSV file as well. This is done by iterating through the properties of the object and then altering data based on conditions we define. Here's an example of iterating through the object to make batch changes:

```PowerShell
$var | foreach {
  if ($_.index -eq 1) {
    $_.color = "Changed"
  }
}
```

And here is the new output of `$var`:

```output
griff@vm923762 ~ $var

index color
----- -----
1     Changed
2     blue
3     yellow
4     green
5     orange
```

You'll see in the above output that the the color for row 1 has been changed from `red` to `Changed`.
There are a variety of ways to accomplish this; I just picked one example. Once you've made your edits, all you need to do is then pipe your `$var` into the command to export as a CSV, which we'll talk about below.

We can also make a single change like so:

```PowerShell
($var | Where-Object { $_.index -eq 1 }).color = "Changed"
```

## Exporting CSV Files
In addition to importing and modifying data from CSV files, we can export that data (or other data) out to a CSV file as well. This is accomplished by piping our `$var` object to the `Export-CSV` command:

```Powershell
$var | Export-CSV -Path 'C:\Path\To\Your\Destination.csv' -NoTypeInformation
```

Opening the resulting CSV file (or importing it into PowerShell like we learned earlier), will result in the updated data as shown here:

| index | color   |
| ----- | ------- |
| 1     | Changed |
| 2     | blue    |
| 3     | yellow  |
| 4     | green   |
| 5     | orange  |

I want to specifically point out the `-NoTypeInformation` parameter that we specified in the above command. If you're using Windows PowerShell (which is by default version 5.1 as bundled with Windows), then you'll want to specify this parameter which suppresses outputting the TYPE information header into your CSV file. By doing this, the first row of cells in your CSV file will actually be the table headers. If you do not specify this, then you get a row above your table headers that looks like this:

```output
#TYPE Selected.System.Management.ManagementObject
```

If I had to guess, I'd say most people most of the time probably don't want this. Luckily, beginning in PowerShell 6, this has become the default so if you're using PowerShell 6 or PowerShell 7 you can skip adding this parameter, though it won't hurt if you continue to include it.

# Working with Excel Files
Now we know how to use the inbuilt commands in PowerShell to work with CSV files, but XLSX files are just as common. Unfortunately there is no native handling of this file format in PowerShell, however there is a third-party module that was developed that does a great job of allowing us to accomplish this; the module is the **ImportExcel** module.

You can install this module and then import it using the following command:

```PowerShell
Install-Module -Name 'ImportExcel' -Force
Import-Module -Name 'ImportExcel' -Force
```

## Importing Excel Files
Importing an XLSX file is quite similar in terms of the command and syntax, just using a different module:

```PowerShell
Import-Excel -Path 'C:\Path\To\Your\File.xlsx'
```

Again, you'll often want to assign this to a variable so you can easily recall it and interact with it. I'll use `$var` again:

```PowerShell
$var = Import-Excel -Path 'C:\Path\To\Your\File.xlsx'
```

In another similarity, here is what the output of `$var` looks like in this scenario:

```output
griff@vm923762 ~  $var = Import-Excel -Path 'C:\temp\excel.xlsx'
griff@vm923762 ~  $var

index color
----- -----
 1.00 red
 2.00 blue
 3.00 yellow
 4.00 green
 5.00 orange
```

Notice that in this instance, importing an Excel workbook instead of a comma-separated values file, that the values of each `index` are doubles instead of integers. While this is the case, if you select only that property of the object using `$var.index`, you'll see that the `.00` does not print to the terminal any longer --  beware it is still a double even though it doesn't look like it. You can validate this by testing `$var.index[0] -is [double]` if you want. This won't prove to be an issue for many people, but for those instances where it is, you could convert the value for each iteration of `index` inside the object to a string, which then allows you to manipulate those values with convenient operators such as `.split` and `.substring`. But I digress.

For the most part, it works pretty much the same as with importing CSVs. There is some additional functionality, such as the ability to import specific worksheets from the workbook using the `-Worksheet` parameter, or filter which columns/rows you'd like to import using `-StartRow`, `-EndRow`, `-StartColumn`, and `-EndColumn` as well as other options.

## Modifying Excel Files
We can use the same method we used earlier to modify the object we imported from the Excel file.
For this example, let's increment the index of each row by 1:

```PowerShell
$var | foreach {
  if ($_.index) {
    $_.index = $_.index + 1.00
  }
}
```

The successful output of our modified `$var` is now:

```output
griff@vm923762 ~ $var

index color
----- -----
 2.00 red
 3.00 blue
 4.00 yellow
 5.00 green
 6.00 orange
```

Again, we can change a single value like so:

```PowerShell
($var | Where-Object { $_.index -eq 1 }).color = "Changed"
```

Just like with our CSV example earlier, now to export it to Excel, you'll pipe your `$var` to the command to export as an Excel file, which we'll look at below.

## Exporting Excel Files
To export your object to an XLSX file:

```PowerShell
Export-Excel -Path 'C:\temp\output2.xlsx' -InputObject $var
```

Alternatively, you can pipe `$var` to the `Export-Excel` command.

```PowerShell
$var | Export-Excel -Path 'C:\temp\output2.xlsx'
```

There are *a lot* of optional parameters for this command, and I highly recommend checking out the module's documentation, which can be found the in Additional Resources section at the end of this post. You can create pivot tables, name worksheets, use multiple worksheets, and all kinds of nifty things!

Opening our exported XLSX file (or importing it into the terminal as we learned earlier), shows the following:

| index | color  |
| ----- | ------ |
| 2     | red    |
| 3     | blue   |
| 4     | yellow |
| 5     | green  |
| 6     | orange |

# Conclusion
And here we are! 

There are other methods of interacting with comma-separated values files and Excel files, some of which are cleaner and more robust as well. The advantage to the methods in this post is that it brings the data into PowerShell as a basic PSCustomObject that you can manipulate pretty much like you would anything else. The disadvantages to this is that these methods lack features that are available in far better and more robust solutions such as, and also lacks the ability to scale nicely as you need to make more and more interactions with the files; if your needs are more complex or you're looking for a more tenable solution, I highly recommend checking out the **PSWriteOffice** project on GitHub! 

In this post we looked at how to work with CSV and XLSX files in PowerShell, from importing them, modifying them, and exporting them as well. With these skills, you can automate and simplify your day-to-day data-related tasks and level up your checks and reporting. I hope that you found this blog post helpful!

# Additional Resources
- [Import-Csv (Microsoft.PowerShell.Utility) - PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/import-csv?view=powershell-7.3)
- [Export-Csv (Microsoft.PowerShell.Utility) - PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/export-csv?view=powershell-7.3)
- [Introducing the PowerShell Excel Module - Scripting Blog (microsoft.com)](https://devblogs.microsoft.com/scripting/introducing-the-powershell-excel-module-2/)
- [GitHub - dfinke/ImportExcel: PowerShell module to import/export Excel spreadsheets, without Excel](https://github.com/dfinke/ImportExcel)
- [GitHub - EvotecIT/PSWriteOffice: Experimental PowerShell Module to create and edit Microsoft Word, Microsoft Excel, and Microsoft PowerPoint documents without having Microsoft Office installed.](https://github.com/EvotecIT/PSWriteOffice)
