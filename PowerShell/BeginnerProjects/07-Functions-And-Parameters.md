# Project 7: Functions and Parameters

## What you'll learn
How to package reusable code into functions, so you write it once and use it everywhere.

## Why this matters
Without functions, every script becomes a long wall of repeated code. Functions let you build your own custom "cmdlets" — reusable, named, and reliable.

## Core concept: Basic function
```powershell
function Say-Hello {
    Write-Host "Hello there!"
}

Say-Hello   # calling the function
```

## Core concept: Parameters
Functions become powerful when they accept input:
```powershell
function Greet-Person {
    param(
        [string]$Name
    )
    Write-Host "Hello, $Name! Welcome."
}

Greet-Person -Name "Alex"
```

## Core concept: Return values
```powershell
function Add-Numbers {
    param(
        [int]$a,
        [int]$b
    )
    return $a + $b
}

$result = Add-Numbers -a 5 -b 3
Write-Host "Result: $result"
```

## Core concept: Default values and multiple parameters
```powershell
function Show-Profile {
    param(
        [string]$Name,
        [int]$Age = 18   # default value if not provided
    )
    Write-Host "$Name is $Age years old."
}

Show-Profile -Name "Sam"          # uses default age
Show-Profile -Name "Priya" -Age 30
```

## The Project: Build a "Toolbox" script with reusable functions
1. Create `toolbox.ps1`.
2. Write a temperature converter function:
   ```powershell
   function Convert-CelsiusToFahrenheit {
       param([double]$Celsius)
       $fahrenheit = ($Celsius * 9/5) + 32
       return $fahrenheit
   }

   $result = Convert-CelsiusToFahrenheit -Celsius 25
   Write-Host "25C = $result F"
   ```
3. Write a function that checks if a number is prime:
   ```powershell
   function Test-IsPrime {
       param([int]$Number)
       if ($Number -lt 2) { return $false }
       for ($i = 2; $i -lt $Number; $i++) {
           if ($Number % $i -eq 0) { return $false }
       }
       return $true
   }

   foreach ($n in 1..20) {
       if (Test-IsPrime -Number $n) {
           Write-Host "$n is prime"
       }
   }
   ```
3. Write a function that greets people differently based on time of day:
   ```powershell
   function Get-Greeting {
       param([string]$Name)
       $hour = (Get-Date).Hour
       if ($hour -lt 12) {
           return "Good morning, $Name!"
       } elseif ($hour -lt 18) {
           return "Good afternoon, $Name!"
       } else {
           return "Good evening, $Name!"
       }
   }

   Write-Host (Get-Greeting -Name "Alex")
   ```

## Challenge
Write a function `Get-BMI` that takes `-WeightKg` and `-HeightM` as parameters and returns the calculated BMI (formula: weight / (height * height)). Test it with your own numbers.

## Key takeaways
- `function Name { ... }` defines reusable code blocks.
- `param()` declares inputs; `[string]`, `[int]`, `[double]` set expected types.
- `return` sends a value back so you can store it in a variable or use it elsewhere.
