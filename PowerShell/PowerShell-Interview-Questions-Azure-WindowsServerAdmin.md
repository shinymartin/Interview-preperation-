# PowerShell Interview Questions & Answers
### For Azure & Windows Server Administration Roles

This document contains 100 real-world PowerShell interview questions, arranged from **basic → intermediate → advanced**, split into role-relevant subsections: core PowerShell, scripting, Windows Server administration, Active Directory, and Azure PowerShell (Az module).

---

## Table of Contents

1. [Section 1: PowerShell Fundamentals (Q1–20)](#section-1-powershell-fundamentals-q1-20)
2. [Section 2: Variables, Data Types & Operators (Q21–30)](#section-2-variables-data-types--operators-q21-30)
3. [Section 3: Pipeline, Objects & Filtering (Q31–42)](#section-3-pipeline-objects--filtering-q31-42)
4. [Section 4: Scripting, Functions & Modules (Q43–58)](#section-4-scripting-functions--modules-q43-58)
5. [Section 5: Windows Server Administration (Q59–72)](#section-5-windows-server-administration-q59-72)
6. [Section 6: Active Directory with PowerShell (Q73–84)](#section-6-active-directory-with-powershell-q73-84)
7. [Section 7: Azure PowerShell (Az Module) (Q85–96)](#section-7-azure-powershell-az-module-q85-96)
8. [Section 8: Advanced / Scenario & Troubleshooting (Q97–100)](#section-8-advanced--scenario--troubleshooting-q97-100)

---

## Section 1: PowerShell Fundamentals (Q1–20)

**1. What is PowerShell?**
PowerShell is a cross-platform task automation and configuration management framework from Microsoft, consisting of a command-line shell, a scripting language built on .NET, and a set of cmdlets used to manage systems, including Windows Server and Azure resources.

**2. How is PowerShell different from Command Prompt (CMD)?**
CMD processes plain text output, while PowerShell processes objects (.NET objects). This means output from one PowerShell command can be piped into another and its properties/methods accessed directly, rather than parsing text.

**3. What is a cmdlet?**
A cmdlet (pronounced "command-let") is a lightweight, single-function command built into PowerShell that follows a `Verb-Noun` naming convention, e.g., `Get-Process`, `Set-Item`.

**4. What is the naming convention for cmdlets and why does it matter?**
Cmdlets use `Verb-Noun` format (e.g., `Get-Service`, `Stop-Process`). This consistency makes commands predictable and discoverable — you can guess `Restart-Service` exists because `Restart` is a standard approved verb.

**5. How do you find the approved list of verbs in PowerShell?**
Using `Get-Verb`, which lists all approved verbs and their intended usage categories (Common, Data, Lifecycle, Security, etc.).

**6. What is the difference between PowerShell 5.1 and PowerShell 7?**
PowerShell 5.1 is built on .NET Framework, is Windows-only, and ships with Windows. PowerShell 7 (PowerShell Core lineage) is built on .NET Core/.NET, is cross-platform (Windows/Linux/macOS), and has better performance and parallel processing (`ForEach-Object -Parallel`).

**7. How do you check the installed PowerShell version?**
```powershell
$PSVersionTable.PSVersion
```

**8. What is `Get-Help` used for?**
It displays help documentation for cmdlets, functions, and concepts. Example: `Get-Help Get-Process -Full` or `Get-Help about_Functions`.

**9. How do you update the local help files?**
```powershell
Update-Help -Force
```

**10. What is `Get-Command` used for?**
It lists all available cmdlets, functions, aliases, and executables recognized by the current session. Example: `Get-Command -Noun Service`.

**11. What are aliases in PowerShell? Give examples.**
Aliases are shortcuts for cmdlet names, e.g., `ls` and `dir` both point to `Get-ChildItem`, `gps` points to `Get-Process`, `?` points to `Where-Object`.

**12. How do you find what cmdlet an alias points to?**
```powershell
Get-Alias -Name ls
```

**13. What is the PowerShell execution policy?**
It's a safety feature that controls whether/how PowerShell scripts (`.ps1` files) are allowed to run. It is not a security boundary but a safeguard against accidental execution.

**14. What are the different execution policy levels?**
- `Restricted` – no scripts allowed (default on client OS)
- `AllSigned` – scripts must be digitally signed
- `RemoteSigned` – local scripts run freely, downloaded scripts must be signed
- `Unrestricted` – all scripts run, with a warning for downloaded ones
- `Bypass` – nothing is blocked, no warnings

**15. How do you check and set the execution policy?**
```powershell
Get-ExecutionPolicy
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**16. What is the PowerShell ISE, and how does it differ from VS Code?**
PowerShell ISE is a legacy built-in GUI editor/debugger for Windows PowerShell (5.1 only, deprecated). VS Code with the PowerShell extension is the modern cross-platform, cross-version editor recommended by Microsoft going forward.

**17. What is a PowerShell profile?**
A profile (`$PROFILE`) is a script that runs automatically when a PowerShell session starts, typically used to set aliases, functions, and environment customizations.

**18. How many profile files can exist, and what's the scope difference?**
There are four: Current User–Current Host, Current User–All Hosts, All Users–Current Host, All Users–All Hosts. They differ by whether they apply to just you or all users, and just the console host or all hosts (ISE, VS Code, etc.).

**19. How do you run a PowerShell script from the command line?**
```powershell
.\script.ps1
# or explicitly
powershell.exe -File .\script.ps1
```

**20. Why might `.\script.ps1` fail even though the file exists?**
Usually due to execution policy restrictions, or because the current directory isn't in the PATH — PowerShell requires the `.\` prefix to run scripts from the current folder for security reasons.

---

## Section 2: Variables, Data Types & Operators (Q21–30)

**21. How do you declare a variable in PowerShell?**
```powershell
$name = "Server01"
```
No explicit type declaration is required; PowerShell is dynamically typed by default.

**22. How do you enforce a specific data type on a variable?**
```powershell
[int]$count = 5
[string]$name = "Test"
```

**23. What are common PowerShell data types?**
`[string]`, `[int]`, `[double]`, `[bool]`, `[datetime]`, `[array]`, `[hashtable]`, `[pscustomobject]`.

**24. What is the difference between an array and a hashtable?**
An array is an ordered collection of values accessed by index (`$arr[0]`). A hashtable is a collection of key-value pairs accessed by key (`$hash["Key"]`).

**25. How do you create a hashtable and loop through it?**
```powershell
$hash = @{ Name = "Server01"; Role = "WebServer" }
foreach ($key in $hash.Keys) {
    "$key : $($hash[$key])"
}
```

**26. What is `$null` and how do you check for it correctly?**
`$null` represents no value. Best practice is to put `$null` on the left side of a comparison: `if ($null -eq $value)` — this avoids issues when `$value` is an array.

**27. What is the difference between `-eq` and `-match`?**
`-eq` performs an exact equality comparison. `-match` performs a regular expression match and also populates the `$Matches` automatic variable.

**28. What comparison operators does PowerShell use instead of `==`, `!=`, `<`, `>`?**
`-eq`, `-ne`, `-lt`, `-gt`, `-le`, `-ge`, plus `-like` (wildcards) and `-match` (regex).

**29. What's the difference between single and double quotes in PowerShell strings?**
Double quotes allow variable expansion and expression interpolation (`"Hello $name"`), while single quotes treat everything literally, including `$` signs.

**30. What is string formatting/interpolation using subexpressions?**
The `$()` subexpression operator lets you embed expressions inside a string:
```powershell
"Today is $(Get-Date -Format 'dddd')"
```

---

## Section 3: Pipeline, Objects & Filtering (Q31–42)

**31. What is the PowerShell pipeline?**
The pipeline (`|`) passes the output object(s) of one cmdlet directly as input to the next cmdlet, without converting to text in between.

**32. How do you inspect the properties/methods of an object?**
```powershell
Get-Process | Get-Member
```

**33. What is the difference between `Where-Object` and `Select-Object`?**
`Where-Object` filters which objects pass through the pipeline based on a condition. `Select-Object` chooses which properties (or a subset/count of objects) are returned.

**34. Give an example of filtering processes by CPU usage above a threshold.**
```powershell
Get-Process | Where-Object { $_.CPU -gt 100 }
```

**35. What does the `$_` variable represent?**
It represents the current object being processed in the pipeline within a script block (e.g., inside `Where-Object`, `ForEach-Object`).

**36. What's the difference between `ForEach-Object` and the `foreach` statement?**
`ForEach-Object` is a cmdlet that processes objects one at a time as they arrive through the pipeline (streaming, memory-efficient). The `foreach` statement operates on an already-loaded collection in memory and is generally faster for in-memory collections but doesn't stream.

**37. How do you sort and group output?**
```powershell
Get-Process | Sort-Object CPU -Descending
Get-Service | Group-Object Status
```

**38. How do you export command output to CSV?**
```powershell
Get-Process | Export-Csv -Path C:\Reports\process.csv -NoTypeInformation
```

**39. How do you convert PowerShell output to JSON and back?**
```powershell
$json = Get-Process | Select-Object -First 1 | ConvertTo-Json
$obj = $json | ConvertFrom-Json
```

**40. What is `Format-Table` vs `Format-List`, and why should they be used carefully in scripts?**
`Format-Table` displays data in columns; `Format-List` displays data as a vertical list of properties. Both convert objects into formatted display strings, which breaks further pipeline processing — they should be used only at the end of a pipeline, never before further object manipulation.

**41. How do you create a custom object?**
```powershell
$obj = [pscustomobject]@{
    Server = "Srv01"
    Status = "Running"
}
```

**42. What does `Tee-Object` do?**
It sends pipeline output to a file (or variable) and simultaneously passes it along the pipeline, useful for logging while still processing output.

---

## Section 4: Scripting, Functions & Modules (Q43–58)

**43. How do you write a basic function in PowerShell?**
```powershell
function Get-Square {
    param([int]$Number)
    return $Number * $Number
}
```

**44. What is the difference between a function and a cmdlet?**
A cmdlet is compiled .NET code (or an advanced function) shipped as part of a module/binary. A function is script-based code, defined and run interpretively. Advanced functions (using `[CmdletBinding()]`) behave almost identically to cmdlets.

**45. What is `[CmdletBinding()]` and why use it?**
It turns a regular function into an "advanced function," enabling common parameters like `-Verbose`, `-ErrorAction`, `-WhatIf`, and better parameter binding/validation.

**46. How do you make a script parameter mandatory?**
```powershell
param(
    [Parameter(Mandatory=$true)]
    [string]$ServerName
)
```

**47. What is parameter validation, and how is it implemented?**
Validation attributes restrict acceptable input values before the function body even runs, e.g.:
```powershell
[ValidateSet("Start","Stop","Restart")]
[string]$Action

[ValidateRange(1,100)]
[int]$Percentage
```

**48. How do you support pipeline input in a function?**
```powershell
function Get-Info {
    param(
        [Parameter(ValueFromPipeline=$true)]
        [string]$ComputerName
    )
    process {
        "Processing $ComputerName"
    }
}
```

**49. What are `begin`, `process`, and `end` blocks used for?**
They control execution phases for pipeline input: `begin` runs once before any input is processed (setup), `process` runs once per pipeline object, and `end` runs once after all input is processed (cleanup/summary).

**50. What is a PowerShell module?**
A module is a packaged, reusable set of cmdlets, functions, variables, and resources (a `.psm1` script module, a `.psd1` manifest, or a compiled binary module) that can be imported into a session.

**51. How do you import and list modules?**
```powershell
Import-Module ActiveDirectory
Get-Module -ListAvailable
```

**52. What is a module manifest (`.psd1`) used for?**
It describes metadata about a module — version, author, required PowerShell version, exported functions, and dependencies — used for validation and packaging (e.g., publishing to the PowerShell Gallery).

**53. How do you handle errors in PowerShell?**
Using `try/catch/finally` blocks combined with `-ErrorAction Stop` so that non-terminating errors are converted into terminating ones that `catch` can trap:
```powershell
try {
    Get-Item "C:\missing.txt" -ErrorAction Stop
} catch {
    Write-Warning "Error: $($_.Exception.Message)"
} finally {
    Write-Host "Cleanup done"
}
```

**54. What is the difference between a terminating and non-terminating error?**
A terminating error stops execution of the current pipeline/script unless caught. A non-terminating error (the default for most cmdlets) reports the error but continues processing remaining items — it will not be caught by `try/catch` unless `-ErrorAction Stop` is specified.

**55. What is `$Error` and how is it used?**
`$Error` is an automatic variable holding an array of recent error objects from the session, most recent first (`$Error[0]`), useful for post-mortem debugging.

**56. How do you debug a PowerShell script?**
Using breakpoints (`Set-PSBreakpoint`), the `Wait-Debugger`/`Set-PSDebug` cmdlets, `Write-Debug` with `-Debug`, or an IDE debugger (VS Code) with step-through, watch variables, and call stack inspection.

**57. What is `Write-Verbose` vs `Write-Output` vs `Write-Host`?**
`Write-Output` sends objects down the pipeline (proper output). `Write-Host` writes directly to the console only, bypassing the pipeline (not capturable/redirectable in older versions). `Write-Verbose` writes diagnostic messages visible only when `-Verbose` is specified, without polluting standard output.

**58. What are `.ps1`, `.psm1`, and `.psd1` files?**
`.ps1` is a script file, `.psm1` is a script module file, and `.psd1` is a module manifest (data) file describing the module's metadata.

---

## Section 5: Windows Server Administration (Q59–72)

**59. How do you check installed roles/features on Windows Server?**
```powershell
Get-WindowsFeature
```

**60. How do you install a role or feature via PowerShell?**
```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
```

**61. How do you check and restart a service?**
```powershell
Get-Service -Name Spooler
Restart-Service -Name Spooler -Force
```

**62. How do you query and filter the Windows Event Log?**
```powershell
Get-WinEvent -LogName System -MaxEvents 50 |
    Where-Object { $_.LevelDisplayName -eq "Error" }
```

**63. What's the difference between `Get-EventLog` and `Get-WinEvent`?**
`Get-EventLog` is older, limited to classic logs, and deprecated in PowerShell 7+. `Get-WinEvent` is faster, supports the newer ETW-based logs, filtering via XPath/FilterHashtable, and works with modern applications and services logs.

**64. How do you check disk space using PowerShell?**
```powershell
Get-PSDrive -PSProvider FileSystem
# or
Get-CimInstance Win32_LogicalDisk | Select-Object DeviceID, @{N='FreeGB';E={[math]::Round($_.FreeSpace/1GB,2)}}
```

**65. What is the difference between `Get-WmiObject` and `Get-CimInstance`?**
`Get-WmiObject` is the legacy cmdlet (deprecated, removed in PowerShell 7) that uses DCOM. `Get-CimInstance` is the modern replacement that uses WSMan/WinRM, works cross-platform, and is more firewall-friendly.

**66. How do you remotely manage a server using PowerShell?**
Using PowerShell Remoting (WinRM) via `Enter-PSSession` (interactive) or `Invoke-Command` (non-interactive, batch):
```powershell
Enter-PSSession -ComputerName Srv01
Invoke-Command -ComputerName Srv01,Srv02 -ScriptBlock { Get-Service }
```

**67. How do you enable PowerShell Remoting on a server?**
```powershell
Enable-PSRemoting -Force
```
This configures WinRM, creates a listener, and sets required firewall rules.

**68. What is a PSSession and why use persistent sessions?**
A `PSSession` is a persistent remote connection created via `New-PSSession`. Reusing a persistent session avoids reconnect overhead when running multiple commands and preserves state (variables, imported modules) between them.

**69. How do you check the local firewall rules via PowerShell?**
```powershell
Get-NetFirewallRule | Where-Object { $_.Enabled -eq 'True' }
```

**70. How do you manage scheduled tasks via PowerShell?**
```powershell
Get-ScheduledTask
Register-ScheduledTask -TaskName "Backup" -Trigger $trigger -Action $action
```

**71. How do you check and configure network adapter settings?**
```powershell
Get-NetIPConfiguration
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.0.0.10 -PrefixLength 24 -DefaultGateway 10.0.0.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.0.0.2
```

**72. How would you automate patch management/updates using PowerShell?**
Using the `PSWindowsUpdate` module (community) or `Windows Update` via WSUS integration:
```powershell
Install-Module PSWindowsUpdate -Force
Get-WindowsUpdate
Install-WindowsUpdate -AcceptAll -AutoReboot
```

---

## Section 6: Active Directory with PowerShell (Q73–84)

**73. How do you import the Active Directory module?**
```powershell
Import-Module ActiveDirectory
```
(Requires RSAT tools installed, or run directly on a Domain Controller.)

**74. How do you create a new AD user via PowerShell?**
```powershell
New-ADUser -Name "John Doe" -SamAccountName jdoe `
  -UserPrincipalName jdoe@contoso.com -Path "OU=Users,DC=contoso,DC=com" `
  -AccountPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) `
  -Enabled $true
```

**75. How do you find disabled or locked-out AD user accounts?**
```powershell
Search-ADAccount -AccountDisabled
Search-ADAccount -LockedOut
```

**76. How do you bulk-create AD users from a CSV file?**
```powershell
Import-Csv .\users.csv | ForEach-Object {
    New-ADUser -Name $_.Name -SamAccountName $_.SamAccountName `
      -Path $_.OU -Enabled $true
}
```

**77. How do you add/remove a user from an AD group?**
```powershell
Add-ADGroupMember -Identity "IT-Admins" -Members jdoe
Remove-ADGroupMember -Identity "IT-Admins" -Members jdoe -Confirm:$false
```

**78. How do you find all members of an AD group, including nested groups?**
```powershell
Get-ADGroupMember -Identity "IT-Admins" -Recursive
```

**79. How do you find AD accounts with passwords about to expire?**
```powershell
Get-ADUser -Filter * -Properties PasswordExpired,PasswordLastSet |
    Where-Object { $_.PasswordExpired -eq $false }
```

**80. How do you reset a user's password and force change at next logon?**
```powershell
Set-ADAccountPassword -Identity jdoe -NewPassword (ConvertTo-SecureString "NewP@ss1" -AsPlainText -Force) -Reset
Set-ADUser -Identity jdoe -ChangePasswordAtLogon $true
```

**81. How do you query computer objects in AD, e.g., all servers?**
```powershell
Get-ADComputer -Filter { OperatingSystem -like "*Server*" } -Properties OperatingSystem
```

**82. How do you find inactive computer/user accounts (not logged in for X days)?**
```powershell
$cutoff = (Get-Date).AddDays(-90)
Get-ADComputer -Filter { LastLogonTimestamp -lt $cutoff } -Properties LastLogonTimestamp
```

**83. What's the difference between `-Filter` and `-LDAPFilter` in AD cmdlets?**
`-Filter` uses PowerShell-style expression syntax (easier to read/write). `-LDAPFilter` uses raw LDAP query syntax, useful for complex queries or when porting existing LDAP filters directly.

**84. How do you move an object between Organizational Units (OUs)?**
```powershell
Get-ADUser jdoe | Move-ADObject -TargetPath "OU=NewOU,DC=contoso,DC=com"
```

---

## Section 7: Azure PowerShell (Az Module) (Q85–96)

**85. What is the Az module, and how is it different from AzureRM?**
`Az` is the current, actively developed PowerShell module for managing Azure resources, replacing the deprecated `AzureRM` module. `Az` also works cross-platform (PowerShell 7+ on Linux/macOS) and coexists with Azure CLI more cleanly.

**86. How do you install the Az module?**
```powershell
Install-Module -Name Az -Scope CurrentUser -Repository PSGallery -Force
```

**87. How do you authenticate to Azure using PowerShell?**
```powershell
Connect-AzAccount
```
For automation/non-interactive scenarios, a service principal is used:
```powershell
Connect-AzAccount -ServicePrincipal -ApplicationId $appId -TenantId $tenantId -CertificateThumbprint $thumb
```

**88. How do you switch between multiple Azure subscriptions?**
```powershell
Get-AzSubscription
Set-AzContext -SubscriptionId "xxxx-xxxx-xxxx"
```

**89. How do you create a resource group and a VM using PowerShell?**
```powershell
New-AzResourceGroup -Name "RG-Prod" -Location "EastUS"

New-AzVM -ResourceGroupName "RG-Prod" -Name "VM01" `
  -Location "EastUS" -Image "Win2022Datacenter" `
  -Credential (Get-Credential)
```

**90. How do you start, stop, and deallocate an Azure VM?**
```powershell
Start-AzVM -ResourceGroupName "RG-Prod" -Name "VM01"
Stop-AzVM -ResourceGroupName "RG-Prod" -Name "VM01" -Force   # deallocates by default
```

**91. Why is "Stopped" vs "Deallocated" state important for Azure VMs?**
A "Stopped" VM (via OS shutdown) still reserves compute capacity and continues to incur compute charges. A "Deallocated" VM (via `Stop-AzVM`) releases the compute resources back to Azure, stopping compute billing — only storage costs remain.

**92. How do you list all resources in a subscription/resource group?**
```powershell
Get-AzResource
Get-AzResource -ResourceGroupName "RG-Prod"
```

**93. How do you deploy an ARM template or Bicep file via PowerShell?**
```powershell
New-AzResourceGroupDeployment -ResourceGroupName "RG-Prod" `
  -TemplateFile ".\main.bicep" -TemplateParameterFile ".\params.json"
```

**94. How do you manage Azure AD (Entra ID) users/groups from PowerShell?**
Using the `Microsoft.Graph` module (current standard, since AzureAD/MSOnline are deprecated):
```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All"
New-MgUser -DisplayName "Jane Doe" -UserPrincipalName "jane@contoso.com" ...
Get-MgGroup -Filter "displayName eq 'IT-Admins'"
```

**95. How do you assign an RBAC role to a user on an Azure resource?**
```powershell
New-AzRoleAssignment -SignInName "jane@contoso.com" `
  -RoleDefinitionName "Contributor" `
  -ResourceGroupName "RG-Prod"
```

**96. How do you automate Azure tasks on a schedule (e.g., nightly VM shutdown)?**
Typically via an Azure Automation Account with a PowerShell Runbook, using a Managed Identity for authentication, triggered by a schedule:
```powershell
Connect-AzAccount -Identity
Get-AzVM -ResourceGroupName "RG-Prod" | Stop-AzVM -Force -AsJob
```

---

## Section 8: Advanced / Scenario & Troubleshooting (Q97–100)

**97. A script works interactively but fails when run as a scheduled task. What are common causes and how do you troubleshoot?**
Common causes: the task runs under a different user context with different permissions/profile; relative paths that assume a specific working directory; execution policy restrictions for that account; missing environment variables (e.g., PATH, `$PROFILE` not loaded because scheduled tasks run non-interactively). Troubleshooting: run the exact same command line used by the task manually as that service account, use full/absolute paths, log output to a transcript (`Start-Transcript`), and check the Task Scheduler history/last run result code.

**98. How would you write a script that safely processes 10,000 remote servers without one bad server halting the whole run?**
Use `Invoke-Command` with `-ThrottleLimit` for parallelism, wrap logic in `try/catch` per target (or use `-ErrorAction SilentlyContinue`/`-ErrorVariable` on the invocation), collect failures into a separate results array/log instead of letting exceptions stop the loop, and consider `ForEach-Object -Parallel` (PowerShell 7+) for large-scale parallel execution with a controlled throttle.

**99. How do you secure credentials in a PowerShell script instead of hardcoding passwords?**
Avoid plaintext passwords entirely. Options: `Get-Credential` for interactive use; `Export-CliXml`/`Import-CliXml` with `ConvertTo-SecureString`/`ConvertFrom-SecureString` (DPAPI-encrypted, tied to user/machine) for saved automation credentials; a secrets vault via the `Microsoft.PowerShell.SecretManagement` module backed by Azure Key Vault; or, in Azure, a Managed Identity so no credential is stored at all.

**100. How would you performance-tune a PowerShell script that is running too slowly against a large dataset (e.g., 50,000 AD objects)?**
Key techniques: avoid `+=` on arrays in a loop (it recreates the array each time) — use a `List<T>` or output directly to the pipeline instead; filter as early as possible at the source (`-Filter`/`-LDAPFilter` on the cmdlet rather than piping to `Where-Object` afterward); request only needed properties (`-Properties` explicitly, not `*`); avoid unnecessary `Format-*` cmdlets mid-pipeline; use `ForEach-Object -Parallel` or runspaces for CPU/network-bound parallelizable work; and measure with `Measure-Command` to isolate the actual bottleneck before optimizing blindly.

---

### Tips for the Interview
- Be ready to **write code live** — practice the syntax for `try/catch`, functions with parameters, and pipeline filtering from memory.
- Expect scenario questions blending **Windows Server + AD + Azure** together (e.g., "onboard a new hire": create AD user, assign groups, provision an Azure VM/mailbox).
- Know **why** you'd choose one cmdlet/module over another (e.g., `Get-CimInstance` vs `Get-WmiObject`, `Az` vs `AzureRM`, `Microsoft.Graph` vs deprecated `AzureAD`).
- Be comfortable explaining **execution policy**, **remoting**, and **credential security** — these come up in almost every admin-focused interview.

---
*Good luck with your interview!*
