# Project 1: Getting Started with PowerShell

## What you'll learn
What PowerShell actually is, how to open it, and the one habit (`Get-Help`) that lets you teach yourself everything else.

## Why this matters
PowerShell isn't just "the black window that runs commands." It's an object-oriented shell — meaning commands pass real data (like tables and lists) to each other, not just plain text. Understanding this early makes everything else click faster.

## Setup
1. Press `Win`, type `PowerShell`, and open **Windows PowerShell** (blue icon) or **PowerShell 7** if you see it (either is fine for this course).
2. You'll see a prompt like `PS C:\Users\YourName>`. That's where you type commands.

## Core concept: Cmdlets
PowerShell commands are called **cmdlets** (pronounced "command-lets"). They follow a strict `Verb-Noun` pattern:
- `Get-Process` → get running processes
- `Get-Service` → get system services
- `Stop-Process` → stop a process

This naming convention means you can often *guess* the right command.

## The Project
Do each step in your PowerShell window and note the output.

1. **Check your PowerShell version:**
   ```powershell
   $PSVersionTable
   ```
2. **See what verbs exist:**
   ```powershell
   Get-Verb
   ```
3. **Find commands about "process":**
   ```powershell
   Get-Command -Noun Process
   ```
4. **Learn about a cmdlet without leaving the shell:**
   ```powershell
   Get-Help Get-Process
   Get-Help Get-Process -Examples
   Get-Help Get-Process -Full
   ```
5. **List running processes and services:**
   ```powershell
   Get-Process
   Get-Service
   ```
6. **Get your computer's basic info:**
   ```powershell
   Get-ComputerInfo | Select-Object CsName, OsName, OsVersion
   ```

## Challenge
Use `Get-Command -Verb Stop` to list every "Stop" cmdlet available on your system. Then use `Get-Help` on one you don't recognize and explain in your own words what it does.

## Key takeaways
- Cmdlets are `Verb-Noun`. Guess first, confirm with `Get-Command`.
- `Get-Help <cmdlet> -Examples` is your best friend — use it constantly.
- PowerShell ships with thousands of cmdlets; you don't memorize them, you discover them.
