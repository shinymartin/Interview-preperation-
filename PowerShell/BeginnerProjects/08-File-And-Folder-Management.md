# Project 8: File and Folder Management

## What you'll learn
How to use PowerShell to create, read, move, rename, and organize files and folders — one of the most practical, real-world uses of PowerShell on Windows.

## Why this matters
This is where PowerShell stops being "just a scripting language" and becomes a tool that saves you real time — cleaning up a messy Downloads folder, batch-renaming photos, backing up files.

## Core concept: Navigating and viewing
```powershell
Get-Location              # where am I?
Set-Location "C:\Users"   # change directory (like cd)
Get-ChildItem              # list files/folders (like dir or ls)
Get-ChildItem -Recurse     # list everything including subfolders
```

## Core concept: Creating and removing
```powershell
New-Item -Path "C:\Temp\MyFolder" -ItemType Directory
New-Item -Path "C:\Temp\MyFolder\notes.txt" -ItemType File
Remove-Item "C:\Temp\MyFolder\notes.txt"
```

## Core concept: Reading and writing file content
```powershell
Set-Content -Path "notes.txt" -Value "Hello, this is line one."
Add-Content -Path "notes.txt" -Value "This is line two."
Get-Content -Path "notes.txt"
```

## Core concept: Copying, moving, renaming
```powershell
Copy-Item "notes.txt" "notes_backup.txt"
Move-Item "notes_backup.txt" "C:\Temp\notes_backup.txt"
Rename-Item "notes.txt" "my_notes.txt"
```

## The Project: Build a "Folder Organizer"
> ⚠️ Practice this in a throwaway test folder first, e.g. `C:\Temp\PracticeArea`, not your real Documents.

1. Create a practice area:
   ```powershell
   New-Item -Path "C:\Temp\PracticeArea" -ItemType Directory
   Set-Location "C:\Temp\PracticeArea"
   ```
2. Generate some sample files to work with:
   ```powershell
   1..5 | ForEach-Object { New-Item -Path "report$_.txt" -ItemType File }
   1..3 | ForEach-Object { New-Item -Path "photo$_.jpg" -ItemType File }
   ```
3. Create subfolders by file type and move files into them:
   ```powershell
   New-Item -Path "TextFiles" -ItemType Directory
   New-Item -Path "Images" -ItemType Directory

   Get-ChildItem -Filter "*.txt" | Move-Item -Destination "TextFiles"
   Get-ChildItem -Filter "*.jpg" | Move-Item -Destination "Images"
   ```
4. Confirm the result:
   ```powershell
   Get-ChildItem -Recurse
   ```
5. Write a log file listing what happened:
   ```powershell
   $log = "Organized on $(Get-Date)"
   Add-Content -Path "organize_log.txt" -Value $log
   Get-Content "organize_log.txt"
   ```

## Challenge
Write a script that scans a folder and automatically sorts *any* files into subfolders based on their extension (hint: use `Get-ChildItem`, `.Extension`, and a loop).

## Key takeaways
- `Get-ChildItem`, `New-Item`, `Move-Item`, `Copy-Item`, `Remove-Item`, `Rename-Item` cover almost all file tasks.
- Piping (`|`) lets you send a list of files straight into another cmdlet for action.
- Always test file-management scripts in a throwaway folder before running them on real data.
