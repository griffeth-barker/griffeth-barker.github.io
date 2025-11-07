---
title: "Use PowerShell to remove a network printer for all users"
tags:
- powershell
- rng
- fun
---

# Rolling Dice in PowerShell
If you're a lover of tabletop role-playing games and like PowerShell, rolling virtual dice with PowerShell can be a fun exercise in scripting. Today, we're dissecting my `Invoke-DiceRoll` function—a simple way to generate random dice rolls using PowerShell. Let's have a look!
## Desired Output
Before getting too far into the script, I thought about what I wanted the output to look like. It's best practice to return only one item, but I want to show the die type and roll result for each one; it might also be nice to show which unit of the dice was rolled if more than one of the same die type was rolled. Because of this, a `PSCustomObject` seemed to be a decent choice, with each die rolled being a child object.

```PowerShell
# Create the object to be output
$rollResults = [System.Collections.Generic.List[PSObject]]::new()

#...

# Actually outputting the object
Write-Output $rollResults
```
## Parameters
We want to be able to roll different quantities and types of polyhedral dice, so we'll accept an integer for each of the most common dice types (d34, d6, d8, d10, d12, and of course, d20):

```PowerShell
  param(
  [Parameter(ParameterSetName = 'Default')]
  [int]$D4 = 0,
  [Parameter(ParameterSetName = 'Default')]
  [int]$D6 = 0,
  [Parameter(ParameterSetName = 'Default')]
  [int]$D8 = 0,
  [Parameter(ParameterSetName = 'Default')]
  [int]$D10 = 0,
  [Parameter(ParameterSetName = 'Default')]
  [int]$D12 = 0,
  [Parameter(ParameterSetName = 'Default')]
  [int]$D20 = 0
  )
```

And we'll use a hashtable to map the type of die to the parameter so we can output the die type in the output later.

```PowerShell
  [hashtable]$diceTypes = @{
    d4  = $D4
    d6  = $D6
    d8  = $D8
    d10 = $D10
    d12 = $D12
    d20 = $D20
  }
```

Now we just need to roll the dice and add the results to `$rollResults`!
## Get-Random and Outputting Results
As we may want to roll multiple dice, we'll use a `foreach` loop. Inside the loop, we'll evaluate what types and quantities are being rolled, then use the `Get-Random` cmdlet to "roll" the dice. The `-Minimum` parameter will be `1`, as it is impossible to roll a `0` on any of our dice, and the `-Maximum` will be the number of sides on the die, plus `1`. 

```PowerShell
  foreach ($dice in $diceTypes.Keys) {
    [int]$count = $diceTypes[$dice]
    [int]$sides = $dice.Substring(1)
    for ($i = 1; $i -le $count; $i++) {
      [int]$roll = Get-Random -Minimum 1 -Maximum ($sides + 1)
      $rollResults.Add([PSCustomObject]@{
        Die       = $dice
        Unit      = $i
        RollValue = $roll
      })
    }
  }
```

With all of the above, we have a complete function as shown below:
```PowerShell
function Invoke-DiceRoll {
  param(
    [Parameter(ParameterSetName = 'Default')]
    [int]$D4 = 0,
    [Parameter(ParameterSetName = 'Default')]
    [int]$D6 = 0,
    [Parameter(ParameterSetName = 'Default')]
    [int]$D8 = 0,
    [Parameter(ParameterSetName = 'Default')]
    [int]$D10 = 0,
    [Parameter(ParameterSetName = 'Default')]
    [int]$D12 = 0,
    [Parameter(ParameterSetName = 'Default')]
    [int]$D20 = 0
  )

  [hashtable]$diceTypes = @{
    d4  = $D4
    d6  = $D6
    d8  = $D8
    d10 = $D10
    d12 = $D12
    d20 = $D20
  }

  $rollResults = [System.Collections.Generic.List[psobject]]::new()

  foreach ($dice in $diceTypes.Keys) {
  [int]$count = $diceTypes[$dice]
    [int]$sides = $dice.Substring(1)
    for ($i = 1; $i -le $count; $i++) {
      [int]$roll = Get-Random -Minimum 1 -Maximum ($sides + 1)
      $rollResults.Add(
        [PSCustomObject]@{
          Die       = $dice
          Unit      = $i
          RollValue = $roll
        }
      )
    }
  }

  Write-Output $rollResults
}
```

And we can roll dice like this:

```PowerShell
# dot source the function, then run the function
. /path/to/Invoke-DiceRoll.ps1
Invoke-DiceRoll -D6 2 -D20 1

# Or, simply call the function script itself
& /path/to/Invoke-DiceRoll.ps1 -D6 2 -D20 1

# Sample output:
# Die Unit RollValue
# --- ---- ---------
# d20    1        11
# d6     1         5
# d6     2         2
```

## Checking Our Work and Wrapping Up
Now that the function is--well--functional...let's check out work. Running the function does not appear to produce any errors, but how well is the randomness working?

I ran the following to see how balanced the dice were, testing using 10,000 d4 dice (which only took 321ms, by the way):

```PowerShell
Invoke-DiceRoll -d4 10000 | 
  Group-Object RollValue | 
  Sort-Object Name -Descending | 
  Select-Object Name, Count

# Output
# Name Count
# ---- -----
# 4     2409
# 3     2531
# 2     2561
# 1     2499
```

Note that the 4 sides of the d4 die have roughly equivalent times rolled over 10,000 iterations, representing pretty fair dice. The spread gets even tighter over 100,000 rolls (which takes around 3.5s):
```PowerShell
Invoke-DiceRoll -d4 100000 | 
  Group-Object RollValue | 
  Sort-Object Name -Descending | 
  Select-Object Name, Count

# Output
# Name Count
# ---- -----
# 4    24984
# 3    24932
# 2    24965
# 1    25119
```

The full function file is available in a [GitHub gist](https://gist.github.com/griffeth-barker/5183c8aedeaab1fe049538f37cd23092).

So there we go! A simple function in PowerShell to "roll" polyhedral dice. Now at your next Dungeons and Dragons session you can be *extra* nerdy and whip out your preferred terminal on your laptop if you've forgotten your dice.  If you do, feel free to share a screenshot or picture of your usage by tagging me in a post on [BlueSky (@griff.systems)](https://griff.systems/bluesky) or [LinkedIn (@griffeth-barker)](https://griff.systems/linkedin)!
