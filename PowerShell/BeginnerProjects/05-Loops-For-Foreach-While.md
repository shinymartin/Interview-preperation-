# Project 5: Loops (For, ForEach, While)

## What you'll learn
How to repeat actions automatically instead of writing the same code over and over.

## Why this matters
Automation is fundamentally about repetition — renaming 500 files, checking 20 servers, processing every row in a spreadsheet. Loops are the engine behind all of it.

## Core concept: For loop
Use when you know how many times to repeat, or need a counter:
```powershell
for ($i = 1; $i -le 5; $i++) {
    Write-Host "Count: $i"
}
```

## Core concept: ForEach loop
Use when looping through a list of items:
```powershell
$fruits = @("Apple", "Banana", "Cherry")
foreach ($fruit in $fruits) {
    Write-Host "I like $fruit"
}
```

## Core concept: While loop
Use when you don't know how many times you'll loop — you just keep going until a condition becomes false:
```powershell
$count = 1
while ($count -le 3) {
    Write-Host "Loop number $count"
    $count++
}
```

## Loop control: break and continue
```powershell
foreach ($num in 1..10) {
    if ($num -eq 5) { break }      # stops the loop entirely
    if ($num % 2 -eq 0) { continue } # skips to next iteration
    Write-Host $num
}
```

## The Project: Build a "Countdown & List Processor" script
1. Create `loops.ps1`.
2. Build a countdown timer:
   ```powershell
   Write-Host "Starting countdown..."
   for ($i = 5; $i -ge 1; $i--) {
       Write-Host $i
       Start-Sleep -Seconds 1
   }
   Write-Host "Liftoff!"
   ```
3. Process a list of names:
   ```powershell
   $names = @("Sam", "Priya", "Alex", "Jordan")
   foreach ($name in $names) {
       Write-Host "Hello, $name! Welcome to PowerShell."
   }
   ```
4. Use a while loop to simulate a simple retry system:
   ```powershell
   $attempts = 0
   $success = $false

   while (-not $success -and $attempts -lt 3) {
       $attempts++
       Write-Host "Attempt $attempts..."
       if ($attempts -eq 3) {
           $success = $true
           Write-Host "Success on attempt $attempts!"
       }
   }
   ```
5. Combine loop + condition: print only even numbers from 1 to 20:
   ```powershell
   foreach ($n in 1..20) {
       if ($n % 2 -eq 0) {
           Write-Host $n
       }
   }
   ```

## Challenge
Write a loop that goes through numbers 1–100 and prints "Fizz" for multiples of 3, "Buzz" for multiples of 5, and "FizzBuzz" for multiples of both (classic exercise — great for practicing loops + conditionals together).

## Key takeaways
- `for` = you control a counter. `foreach` = you're processing a list. `while` = repeat until a condition changes.
- `break` exits a loop early; `continue` skips to the next round.
- `1..20` is shorthand for a range of numbers — very handy inside loops.
