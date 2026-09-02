# Project 9: Error Handling (Try, Catch, Finally)

## What you'll learn
How to anticipate things going wrong in your scripts, and handle them gracefully instead of letting your script crash.

## Why this matters
Real scripts run into missing files, bad input, no internet connection. Professional scripts don't just crash — they catch the problem and respond sensibly. This is what separates a "script that works on my machine" from one you can actually trust.

## Core concept: Try / Catch / Finally
```powershell
try {
    $result = 10 / 0
} catch {
    Write-Host "Something went wrong: $($_.Exception.Message)"
} finally {
    Write-Host "This always runs, error or not."
}
```

## Core concept: Common places errors happen
- Trying to read a file that doesn't exist
- Dividing by zero
- Converting text to a number when the text isn't a valid number
- Calling a cmdlet with wrong parameters

## Core concept: Throwing your own errors
Sometimes *you* want to stop execution on purpose:
```powershell
function Test-Age {
    param([int]$Age)
    if ($Age -lt 0) {
        throw "Age cannot be negative."
    }
    Write-Host "Age is valid: $Age"
}

try {
    Test-Age -Age -5
} catch {
    Write-Host "Caught an error: $($_.Exception.Message)"
}
```

## The Project: Build a "Safe File Reader" script
1. Create `safereader.ps1`.
2. Try reading a file that may or may not exist:
   ```powershell
   $filePath = Read-Host "Enter a file path to read"

   try {
       $content = Get-Content -Path $filePath -ErrorAction Stop
       Write-Host "File content:"
       Write-Host $content
   } catch {
       Write-Host "Could not read the file. Reason: $($_.Exception.Message)"
   } finally {
       Write-Host "Attempt finished at $(Get-Date)"
   }
   ```
3. Build a safe number converter:
   ```powershell
   $input1 = Read-Host "Enter a number"

   try {
       $number = [int]$input1
       Write-Host "You entered: $number"
   } catch {
       Write-Host "That wasn't a valid whole number. Please try again."
   }
   ```
4. Build a safe division calculator:
   ```powershell
   function Divide-Numbers {
       param([double]$a, [double]$b)
       try {
           if ($b -eq 0) {
               throw "Cannot divide by zero."
           }
           return $a / $b
       } catch {
           Write-Host "Error: $($_.Exception.Message)"
           return $null
       }
   }

   Divide-Numbers -a 10 -b 0
   Divide-Numbers -a 10 -b 2
   ```

## Challenge
Combine Project 8's folder organizer with error handling: wrap the file-moving logic in `try/catch` so that if a file is locked or already exists at the destination, the script reports it instead of crashing.

## Key takeaways
- `try { }` wraps risky code; `catch { }` handles what happens if it fails; `finally { }` always runs.
- `-ErrorAction Stop` forces a cmdlet's error to be "catchable" (many cmdlets don't stop by default).
- `throw` lets you create your own custom errors when something violates your script's rules.
