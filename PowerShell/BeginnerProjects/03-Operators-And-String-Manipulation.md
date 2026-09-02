# Project 3: Operators and String Manipulation

## What you'll learn
How to do math, compare values, and slice/combine text (strings) in PowerShell.

## Why this matters
Almost every script eventually needs to compare two things ("is this file bigger than 1MB?") or reshape text ("turn this name into all caps"). This project builds that muscle memory.

## Core concept: Arithmetic operators
```powershell
5 + 3   # Addition
5 - 3   # Subtraction
5 * 3   # Multiplication
5 / 3   # Division
5 % 3   # Modulus (remainder)
```

## Core concept: Comparison operators
PowerShell uses letter-based comparison operators, not symbols like `==`:
| Operator | Meaning |
|---|---|
| `-eq` | equal to |
| `-ne` | not equal to |
| `-gt` | greater than |
| `-lt` | less than |
| `-ge` | greater or equal |
| `-le` | less or equal |

```powershell
5 -eq 5     # True
5 -gt 10    # False
"abc" -eq "ABC"   # True (strings are case-insensitive by default!)
```

## Core concept: String manipulation
```powershell
$text = "Learning PowerShell"
$text.ToUpper()
$text.ToLower()
$text.Length
$text.Substring(0,8)
$text.Replace("PowerShell", "Scripting")
$text.Split(" ")
"  trim me  ".Trim()
```

## The Project: Build a "Text Toolbox" script
1. Create `texttoolbox.ps1`.
2. Ask for input and manipulate it:
   ```powershell
   $sentence = Read-Host "Type any sentence"

   Write-Host "Uppercase: $($sentence.ToUpper())"
   Write-Host "Lowercase: $($sentence.ToLower())"
   Write-Host "Length: $($sentence.Length) characters"
   Write-Host "Reversed words:"
   $words = $sentence.Split(" ")
   [array]::Reverse($words)
   Write-Host ($words -join " ")
   ```
3. Add a simple calculator section:
   ```powershell
   $num1 = Read-Host "Enter first number"
   $num2 = Read-Host "Enter second number"
   $num1 = [int]$num1
   $num2 = [int]$num2

   Write-Host "$num1 + $num2 = $($num1 + $num2)"
   Write-Host "$num1 - $num2 = $($num1 - $num2)"
   Write-Host "$num1 * $num2 = $($num1 * $num2)"
   if ($num2 -ne 0) {
       Write-Host "$num1 / $num2 = $($num1 / $num2)"
   }
   ```
4. Run the script and try it with different words and numbers.

## Challenge
Add a check: is `$num1` greater than `$num2`? Print "bigger", "smaller", or "equal" using comparison operators (you'll use `if` fully in Project 4 — for now just try one comparison and `Write-Host` the raw `$true`/`$false` result).

## Key takeaways
- Comparison operators are word-based (`-eq`, `-gt`), not symbols.
- `Read-Host` always returns a string — cast it with `[int]` before doing math.
- `$()` inside a string lets you run a mini-expression directly in text output.
