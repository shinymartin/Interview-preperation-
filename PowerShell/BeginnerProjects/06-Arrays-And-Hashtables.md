# Project 6: Arrays and Hashtables

## What you'll learn
How to store and organize multiple pieces of related data — lists (arrays) and key-value pairs (hashtables).

## Why this matters
Real data isn't a single value — it's a list of files, a table of employees, settings with names attached. Arrays and hashtables are how PowerShell organizes that.

## Core concept: Arrays
An array is an ordered list of items.
```powershell
$colors = @("Red", "Green", "Blue")
$colors[0]          # "Red" - arrays start at index 0
$colors.Count       # 3
$colors += "Yellow" # add an item
$colors             # loops through all values when printed
```

## Core concept: Hashtables
A hashtable stores data as **key-value pairs** — like a labeled dictionary.
```powershell
$person = @{
    Name = "Alex"
    Age  = 25
    City = "Chicago"
}
$person["Name"]     # "Alex"
$person.Age         # 25 - dot notation also works
$person["Country"] = "USA"   # add a new key
```

## The Project: Build a "Mini Inventory System"
1. Create `inventory.ps1`.
2. Create an array of products:
   ```powershell
   $products = @("Laptop", "Mouse", "Keyboard", "Monitor")

   Write-Host "We have $($products.Count) products:"
   foreach ($item in $products) {
       Write-Host "- $item"
   }
   ```
3. Turn it into a hashtable with prices:
   ```powershell
   $prices = @{
       Laptop   = 800
       Mouse    = 20
       Keyboard = 45
       Monitor  = 150
   }

   foreach ($item in $prices.Keys) {
       Write-Host "$item costs `$$($prices[$item])"
   }
   ```
4. Calculate the total inventory value:
   ```powershell
   $total = 0
   foreach ($item in $prices.Keys) {
       $total += $prices[$item]
   }
   Write-Host "Total inventory value: `$$total"
   ```
5. Build an array of hashtables (like mini database rows):
   ```powershell
   $employees = @(
       @{ Name = "Sam"; Role = "Developer" },
       @{ Name = "Priya"; Role = "Designer" },
       @{ Name = "Alex"; Role = "Manager" }
   )

   foreach ($emp in $employees) {
       Write-Host "$($emp.Name) works as a $($emp.Role)"
   }
   ```

## Challenge
Add a new product to `$prices` through user input (`Read-Host` for name and price), then reprint the full list with the new total.

## Key takeaways
- Arrays = ordered lists, accessed by index (`$array[0]`).
- Hashtables = key-value pairs, accessed by key (`$hash["key"]` or `$hash.key`).
- An array of hashtables is a simple but powerful way to represent table-like data (rows and columns).
