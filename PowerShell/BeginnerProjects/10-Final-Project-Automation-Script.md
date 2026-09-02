# Project 10: Final Project — Build a System Health & Cleanup Reporter

## What you'll learn
How to combine everything from Projects 1–9 (variables, operators, conditionals, loops, arrays/hashtables, functions, file management, and error handling) into one real, useful automation script.

## Why this matters
This is the project that proves you actually learned PowerShell — not just individual pieces, but how they work together to build something practical you could genuinely use on your own laptop.

## What the script will do
1. Report basic system info (Project 1 concepts)
2. Check disk space and warn if low (Project 4: conditionals)
3. List the top 5 largest files in a folder (Project 5 & 6: loops, arrays)
4. Organize a "Downloads-like" test folder by file type (Project 8: file management)
5. Log everything to a report file, handling any errors gracefully (Project 9: error handling)
6. Use custom functions to keep the code clean (Project 7: functions)

## The Project
Create `SystemReport.ps1` and build it in these stages. Test each stage before moving to the next.

### Stage 1: Basic info + logging function
```powershell
function Write-Log {
    param([string]$Message)
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $line = "[$timestamp] $Message"
    Write-Host $line
    Add-Content -Path "SystemReport.log" -Value $line
}

Write-Log "=== System Report Started ==="
Write-Log "Computer Name: $env:COMPUTERNAME"
Write-Log "PowerShell Version: $($PSVersionTable.PSVersion)"
```

### Stage 2: Disk space check with conditionals
```powershell
function Check-DiskSpace {
    $disk = Get-PSDrive C
    $freeGB = [math]::Round($disk.Free / 1GB, 2)
    $usedGB = [math]::Round($disk.Used / 1GB, 2)

    Write-Log "C: Drive - Free: $freeGB GB | Used: $usedGB GB"

    if ($freeGB -lt 10) {
        Write-Log "WARNING: Low disk space!"
    } elseif ($freeGB -lt 50) {
        Write-Log "Notice: Disk space getting moderate."
    } else {
        Write-Log "Disk space looks healthy."
    }
}

Check-DiskSpace
```

### Stage 3: Find largest files in a folder (loops + arrays)
```powershell
function Get-TopFiles {
    param([string]$FolderPath, [int]$Top = 5)

    try {
        $files = Get-ChildItem -Path $FolderPath -File -ErrorAction Stop |
                 Sort-Object Length -Descending |
                 Select-Object -First $Top

        Write-Log "Top $Top largest files in $FolderPath :"
        foreach ($file in $files) {
            $sizeMB = [math]::Round($file.Length / 1MB, 2)
            Write-Log " - $($file.Name): $sizeMB MB"
        }
    } catch {
        Write-Log "ERROR scanning folder: $($_.Exception.Message)"
    }
}

Get-TopFiles -FolderPath "$env:USERPROFILE\Downloads" -Top 5
```

### Stage 4: Organize a test folder by extension (file management)
```powershell
function Organize-Folder {
    param([string]$FolderPath)

    try {
        $files = Get-ChildItem -Path $FolderPath -File -ErrorAction Stop
        foreach ($file in $files) {
            $ext = $file.Extension.TrimStart(".").ToUpper()
            if ([string]::IsNullOrEmpty($ext)) { $ext = "NoExtension" }

            $destFolder = Join-Path $FolderPath $ext
            if (-not (Test-Path $destFolder)) {
                New-Item -Path $destFolder -ItemType Directory | Out-Null
            }
            Move-Item -Path $file.FullName -Destination $destFolder -ErrorAction Stop
        }
        Write-Log "Organized folder: $FolderPath"
    } catch {
        Write-Log "ERROR organizing folder: $($_.Exception.Message)"
    }
}

# Create and test with a throwaway folder first!
New-Item -Path "C:\Temp\TestDownloads" -ItemType Directory -Force | Out-Null
1..3 | ForEach-Object { New-Item -Path "C:\Temp\TestDownloads\file$_.txt" -ItemType File }
1..2 | ForEach-Object { New-Item -Path "C:\Temp\TestDownloads\img$_.jpg" -ItemType File }

Organize-Folder -FolderPath "C:\Temp\TestDownloads"
```

### Stage 5: Wrap it all together
```powershell
Write-Log "=== System Report Completed ==="
Write-Host "`nFull report saved to SystemReport.log"
```

## Running the whole thing
1. Save all stages into one file: `SystemReport.ps1`.
2. Run it:
   ```powershell
   .\SystemReport.ps1
   ```
3. Open `SystemReport.log` afterward and review what it captured.

## Challenge (optional, but recommended)
Extend the script to email or display a summary count of how many files were organized into each category, using a hashtable to tally counts as you loop through files.

## Key takeaways
- Real scripts are just Projects 1–9's concepts combined: variables hold data, conditionals make decisions, loops process lists, functions organize logic, and error handling keeps it from crashing.
- Logging (writing progress to a file) is a simple habit that makes any script easier to trust and debug.
- Always test destructive operations (moving/deleting files) on a throwaway folder first — a habit worth keeping forever.

---
### 🎉 Congratulations
If you've completed all 10 projects, you now understand PowerShell's core building blocks: cmdlets, variables, operators, conditionals, loops, arrays/hashtables, functions, file management, and error handling — and how they combine into real automation. From here, the best way to keep learning is to automate something small in your actual daily routine.
