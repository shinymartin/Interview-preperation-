# Project 2: Variables and Data Types

## What you'll learn
How to store and reuse information using variables, and the basic data types PowerShell works with.

## Why this matters
Every script needs to remember things — a name, a number, a file path. Variables are how you store that. Getting comfortable with them is the foundation for every project after this one.

## Core concept: Variables
In PowerShell, every variable starts with `$`. You don't need to declare a type — PowerShell figures it out for you.

```powershell
$name = "Alex"
$age = 25
$isStudent = $true
```

## Core concept: Data types
| Type | Example | Meaning |
|---|---|---|
| String | `"Hello"` | Text |
| Integer | `42` | Whole number |
| Double | `3.14` | Decimal number |
| Boolean | `$true` / `$false` | Yes/No value |
| Array | `@(1,2,3)` | A list of items |

Check any variable's type with:
```powershell
$age.GetType()
```

## The Project: Build a "Personal Profile" script
1. Open a new file in Notepad (or VS Code if you have it) and save it as `profile.ps1`.
2. Add the following, replacing values with your own:
   ```powershell
   $name = "Your Name"
   $age = 22
   $city = "Your City"
   $favoriteLanguages = @("PowerShell", "Python")
   $isLearningPowerShell = $true

   Write-Host "Name: $name"
   Write-Host "Age: $age"
   Write-Host "City: $city"
   Write-Host "Languages: $favoriteLanguages"
   Write-Host "Learning PowerShell right now? $isLearningPowerShell"
   ```
3. Run it. In PowerShell, navigate to the folder with `cd` and run:
   ```powershell
   .\profile.ps1
   ```
   > If you get a "scripts are disabled" error, run this once as Administrator:
   > `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`
4. Add a calculation using your age:
   ```powershell
   $birthYear = 2026 - $age
   Write-Host "You were likely born around $birthYear"
   ```
5. Check the type of each variable:
   ```powershell
   $name.GetType().Name
   $age.GetType().Name
   $favoriteLanguages.GetType().Name
   ```

## Challenge
Add a variable `$monthlyBudget` and calculate your `$dailyBudget` by dividing it by 30. Print a sentence combining both values.

## Key takeaways
- Variables always start with `$` and don't need a declared type.
- PowerShell infers types automatically — but you can always check with `.GetType()`.
- String interpolation (`"Name: $name"`) lets you embed variables directly in text.
