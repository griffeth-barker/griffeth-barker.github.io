---
title: "PowerShell. Properties. Period."
tags:
  - powershell
  - objects
  - pipeline
categories:
  - powershell
---

## Why Properties Are So Strong

If you're a systems administrator or engineer working in the Microsoft ecosystem, you live and breathe data—service statuses, user accounts, disk space, and performance counters. While you're likely familiar with core PowerShell commands, maximizing your efficiency requires a deep understanding of one fundamental concept: **objects and their properties.**

In PowerShell, when you run a command like `Get-Service` or `Get-ADUser`, the output isn't just text on a screen; it's a collection of rich **objects**. These objects are the key to unlocking automation efficiency.

Think of an object like a **structured data form** that describes something (a service, a user, or a process). The properties are the specific **fields** on that form (e.g., Name, Status, StartType, Description).

### The Power of the Object Model

Why is this object-oriented approach superior to old-school text parsing?

1.  **Uniformity and Reliability:** Every object returned by the same cmdlet (e.g., all objects from `Get-ADUser`) has a consistent, named set of properties. This consistency means your scripts will work reliably every time, unlike scripts that try to parse text output, which can break when a column or header changes.
2.  **Selectivity and Precision:** You don't have to deal with a wall of text output. You can pinpoint and extract **exactly** the piece of data you need for reporting, logging, or passing to the next command.
3.  **Actionable Data:** Properties contain the values you often need to **change** (`Set-Service -Status Stopped`, `Set-ADUser -Title "Manager"`). They are the interface for configuration management.

-----

## The Dot Notation: Precision and Speed

The **dot notation** (the period, `.`) is the single most important operator for working with object properties and methods in PowerShell. It allows you to drill down into an object and access its specific members.

### 1\. Selecting a Single Property Value

To extract the value of a specific property from an object, you use the dot operator directly against that object. Notice the use of parentheses to ensure the object is retrieved *before* the dot operator acts on it.

```powershell
# Get the status of the Print Spooler service
(Get-Service -Name Spooler).Status
# Output: Running
```

| Component | Meaning |
| :--- | :--- |
| `(Get-Service -Name Spooler)` | Retrieves the **object** (the Print Spooler service). |
| **`.`** | The **dot operator** (the property accessor). |
| `Status` | The **property name**. |

### 2\. Property Enumeration (Quick Lists)

The standard way to select multiple properties from a collection of objects is using `Select-Object`. However, if you only need a *single* property value from *every* object in a collection, the dot notation offers a cleaner, faster shortcut known as **Property Enumeration**.

When you use the dot operator directly on a collection (like the full list returned by `Get-Service`), PowerShell automatically iterates over the entire collection and returns the value of that property for each item.

```powershell
# Get a simple list of all running service names
(Get-Service | Where-Object {$_.Status -eq 'Running'}).Name
# Output:
# ssh-agent
# Spooler
# gupdatem
# ...
```

This is significantly more concise than piping to `Select-Object -ExpandProperty Name`.

### 3\. Drilling Down with Subproperties

Objects can contain other objects, which is a key concept in PowerShell. You use the dot notation to **chain** property access and retrieve these **subproperties**.

For example, the output of `Get-Process` includes a `MainModule` property, which is itself an object containing details about the process's executable.

```powershell
# Get the full file path for the main executable of the PowerShell process
(Get-Process -Name powershell).MainModule.FileName
```

In this case, we chained the property access: `Object.Property.Subproperty`.

-----

## Dot Operator Methods: Actions and Operations

Beyond properties (the data), objects also contain **methods** (the actions or operations) you can execute. You call a method using the dot notation, followed by the method name and a set of parentheses `()`.

### Example: String Manipulation

When you retrieve a property that contains text (a string), you can leverage built-in string methods immediately.

Suppose you retrieve a user's full name, but you need it converted to all lowercase for use in a path or a log entry:

```powershell
$userName = (Get-ADUser -Identity 'jdoe').Name
# $userName is "John Doe"

# Use the .ToLower() method
$userName.ToLower()
# Output: john doe
```

Other useful string methods include: `.ToUpper()`, `.Trim()`, and `.Replace("old", "new")`.

### Example: Collection Management

Arrays and collections also have powerful methods you can use to manage their contents directly, saving you from complex piping and looping.

```powershell
# Create an array of strings (a collection)
$servers = "ServerA", "ServerB", "ServerC"

# Use the .Remove() method to remove an item directly from the array
$servers.Remove("ServerB")

# Check the remaining list
$servers
# Output:
# ServerA
# ServerC
```

## 💡 How to Discover Properties?

If you don't know what properties or methods an object has, use the **`Get-Member`** cmdlet.

Pipe any object to it, and it will list all available members, categorized as **Properties** (data) or **Methods** (actions):

```powershell
Get-Service -Name Spooler | Get-Member -MemberType Property
```

The output will show the property name and its type (e.g., `Status` is a `ServiceControllerStatus`, and `Name` is a `String`).

## Final Thoughts

Mastering the dot notation and embracing the object model is not just a best practice—it is the foundation of modern, efficient, and robust PowerShell scripting. Stop wrestling with unpredictable text output. Use the properties built into the objects.
