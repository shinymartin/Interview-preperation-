# Project 4: Conditional Logic (If, Else, Switch)

## What you'll learn
How to make your scripts make decisions — doing different things depending on the situation.

## Why this matters
A script that always does the exact same thing isn't very useful. Real automation reacts: "if disk space is low, warn me" or "if the file exists, skip it." Conditionals are how scripts think.

## Core concept: If / ElseIf / Else
```powershell
$temperature = 15

if ($temperature -gt 30) {
    Write-Host "It's hot!"
} elseif ($temperature -gt 15) {
    Write-Host "It's mild."
} else {
    Write-Host "It's cold."
}
```

## Core concept: Logical operators
Combine multiple conditions:
```powershell
-and   # both must be true
-or    # at least one must be true
-not   # flips true/false
```
```powershell
if ($temperature -gt 10 -and $temperature -lt 25) {
    Write-Host "Comfortable weather"
}
```

## Core concept: Switch statement
Useful when you're checking one variable against many possible values:
```powershell
$day = "Monday"

switch ($day) {
    "Saturday" { Write-Host "Weekend!" }
    "Sunday"   { Write-Host "Weekend!" }
    default    { Write-Host "Weekday, get to work" }
}
```

## The Project: Build a "Decision Helper" script
1. Create `decisionhelper.ps1`.
2. Write a simple grading system:
   ```powershell
   $score = Read-Host "Enter your test score (0-100)"
   $score = [int]$score

   if ($score -ge 90) {
       Write-Host "Grade: A"
   } elseif ($score -ge 75) {
       Write-Host "Grade: B"
   } elseif ($score -ge 60) {
       Write-Host "Grade: C"
   } else {
       Write-Host "Grade: F - keep practicing!"
   }
   ```
3. Add a day-type checker using `switch`:
   ```powershell
   $day = Read-Host "Enter today's day (e.g. Monday)"

   switch ($day) {
       "Saturday" { Write-Host "It's the weekend, relax!" }
       "Sunday"   { Write-Host "It's the weekend, relax!" }
       default    { Write-Host "It's a weekday, stay productive." }
   }
   ```
4. Add a combined condition check:
   ```powershell
   $hasLaptop = $true
   $hasInternet = $true

   if ($hasLaptop -and $hasInternet) {
       Write-Host "You're ready to learn PowerShell!"
   } elseif ($hasLaptop -and -not $hasInternet) {
       Write-Host "You can practice offline, but tutorials need internet."
   } else {
       Write-Host "You need a laptop first."
   }
   ```

## Challenge
Extend the grading script so scores below 0 or above 100 print "Invalid score" instead of a grade.

## Key takeaways
- `if / elseif / else` handles ranges and complex logic.
- `switch` is cleaner than many `if/elseif` chains when checking one value against many options.
- `-and`, `-or`, and `-not` let you combine conditions naturally.
