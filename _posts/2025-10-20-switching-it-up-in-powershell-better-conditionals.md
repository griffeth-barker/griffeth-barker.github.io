---
title: "Switching It Up in PowerShell: Better Conditionals"
date: "2025-10-20"
tags:
  - powershell
  - conditionals
categories:
  - powershell
---

## "If this happens, then I want this to happen." 

Many IT professionals have, as part of a PowerShell script, written the code equivalent to this looking to solve a problem. Conditional statements are a key component of automation. But what about when you have multiple possible scenarios? Raise your hand if you've gotten to this point...

```powershell
if ( $option -eq '1' ) {
    Do-Thing1
} 
if ( $option -eq '2' ) {
    Do-Thing2
}
if ( $option -eq '3' ) {
    Do-Thing3
}
# ...and a bunch more of this...
elseif {
    Do-SomethingElse
}
```

...and realized that things are getting pretty unwieldy? I think many of us have. This is when it's time to "switch it up" with the switch statement!
Switch Statements

## Switch Statements
switch is a conditional statement that evaluates a single input and selects the desired matching code block. Let's take the earlier example and make it a switch statement!

```powershell
switch ( $option ) {
    1 {
        Do-Thing1
    }
    2 {
        Do-Thing2
    }
    3 {
        Do-Thing3
    }
    # ... other cases ...
    Default {
        Do-SomethingElse
    }
}
```

There are some visible improvements here in terms of readability, but there are some potential performance benefits as well.

## Performance

In most scripts, the performance benefit of a switch statement is probably unimportant. The if/then statement is performant enough when there are only a couple of possible conditions. But once you scale up to many possible conditions, the repetitive if clauses become very inefficient and the switch becomes the optimal choice due to how the two behave in .NET under-the-hood.  

The main differences:
- If/Else Chain: The script must sequentially evaluate every condition in order until a match is found. If the match is the 10th one, the first 9 checks are wasted effort.  
- Switch Statement: When a switch is used for literal comparisons (checking a value against exact strings, integers, or enums), the compiler often generates a jump table (similar to a hash table lookup). This allows the runtime to go directly to the matching code block in near-constant time, regardless of how many cases you have.  

Imagine you walk into a large business building with instructions to talk to John. You don't know which office John is in, so you start knocking on doors until you find him...in the 51st office. This means that you wasted a whole lot of time checking those other 50 offices. That's somewhat similar to how an if/then chain behaves. Wouldn't you prefer to walk in, look at the building directory, and immediately go to the 63rd office to talk to John? That's more like a switch statement.

This "direct jump" mechanism is why a switch can be significantly faster than a long, chained if structure when processing a single variable against many fixed values.

Unfortunately, the performance benefit of the jump table is often lost when you introduce advanced comparison operators, such as -Regex. When doing this, the matching goes back to happening iteratively.
Quality of Life Improvements

Even when performance is equal, the switch statement offers superior efficiency and readability. Because the target variable is evaluated only once, there is some computational time saved, as well as extra lines of code reduced (as seen above). An if/elseif chain often requires repeating the comparison logic for the same variable multiple times.

Another convenience benefit is that the switch structure separates the condition from the action clearly, eliminating the need for bulky, nested parentheses and keyword repetition.

Finally, a switch can automatically process each element of an array without needing a separate foreach loop to handle it.

## Real World Example

Here's an example of where a switch statement could be helpful. Let's say you have an Active Directory or Entra ID environment and you want to set some properties (maybe phone number and address) of some users based on their department.

```powershell
switch ( $user.Department ) {
    'Accounting' {
        $officeLocation = '123 Dollar Drive'
        $telephone      = '555-1111'
    }
    'Audit' {
        $officeLocation = '456 Compliance Court'
        $telephone      = '555-2222'
    }
    'HR' {
        $officeLocation = '789 Pension Place'
        $telephone      = '555-3333'
    }
    'Legal' {
        $officeLocation = '111 Plaintiff Parkway'
        $telephone      = '555-4444'
    }
    'IT' {
        $officeLocation = '222 Computer Circle'
        $telephone      = '555-5555'
    }
    'Marketing' {
        $officeLocation = '333 Advertising Avenue'
        $telephone      = '555-6666'
    }
    'Sales' {
        $officeLocation = '444 Pitching Place'
        $telephone      = '555-7777'
    }
    'Warehouse' {
        $officeLocation = '555 Shipping Street'
        $telephone      = '555-8888'
    }
}
```

See how we didn't need to say "if the user's department is X" a bunch of times, and how the resulting structure is highly readable? Remember again that the script will be able to jump directly to the matching code block without wasting time checking all of the alternative cases.

## Wrapping Up
In general, if you find yourself chaining together more than 2 conditions, then switch it up! If you want to learn more about switches, check out [about_Switch - PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_switch?view=powershell-7.5).  

How have you used switch statements?