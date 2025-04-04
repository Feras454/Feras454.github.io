---
title: Useful PowerShell Commands for Incident Response (IR) triage
description:
date: 2024-02-07
image: pw.png
license: 
comments: false
draft: false

---





 
 **This post explores some of PowerShell commands that I've found invaluable for rapid triage during investigations.**

---
## System Information

- **System Info**:  

```powershell
Get-ComputerInfo
```

## Process Management

- **Running Processes**:  

```powershell
Get-Process
```

- **Kill Process by Name**:  

```powershell
Stop-Process -Name "NameOfProcess" -Force
```

- **List Running Processes, Parent Process, and Path**:  

```powershell
Get-WmiObject -Class Win32_Process | ForEach-Object {

    $processID = $_.ProcessID

    $parentProcessID = $_.ParentProcessID

    $parentProcess = Get-WmiObject -Class Win32_Process -Filter "ProcessID='$parentProcessID'"

    [PSCustomObject]@{

        ProcessName = $_.Name

        ProcessID = $processID

        ParentProcessName = $parentProcess.Name

        ExecutablePath = $_.ExecutablePath

        CommandLine = $_.CommandLine

    }

}
```

- **Generate Hashes of All Running Processes**:  

```powershell
Get-Process | ForEach-Object {

    try {

        $hash = Get-FileHash $_.Path -Algorithm SHA256

        Write-Output "$($_.ProcessName), $hash"

    }

    catch {

        Write-Output "$($_.ProcessName), $_.Exception.Message"

    }

}
```

  

## Network Information

- **Network Connections**:  

```powershell
Get-NetTCPConnection
```

- **Get Listening Ports**:  

```powershell
Get-NetTCPConnection -State Listen | Select-Object LocalAddress, LocalPort, OwningProcess, OwningProcessName
```

- **ARP Table**:  

```powershell
Get-NetNeighbor
```

```powershell
Get-NetNeighbor | Where-Object {$_.State -eq 'Reachable'} | Format-Table IPAddress, LinkLayerAddress, State
```

- **Check for Open RDP Sessions**
```powershell
Get-WmiObject -Class Win32_Session | Where-Object {$_.SessionType -eq "RDP"}

```

- **Check DNS Cache for Signs of DNS Hijacking**
```powershell
Get-DnsClientCache
```

- **Get Active Network Shares**
```powershell
Get-SmbShare
```
## File System and Registry

- **Traverse Filesystem**:  

```powershell

Get-ChildItem [file] | Format-List *

```

- **Get File Metadata**:  

```powershell

Get-ItemProperty C:\Test\Weather.xls | Format-List

```

- **File Hash Calculation**:  

```powershell

Get-FileHash -Algorithm MD5 C:\path\to\file

```

- **Find Large Files**:  

```powershell

gci -r | sort -descending -property length | select -first 10 name, length, directory

```

- **Files Created or Modified Between Two Dates**:  

```powershell

Get-ChildItem -Path "C:\Path\to\folder" -Recurse | Where-Object {($_.CreationTime -ge (Get-Date "2023-06-11")) -and ($_.CreationTime -le (Get-Date "2023-06-22")) -or ($_.LastWriteTime -ge (Get-Date "2023-06-11")) -and ($_.LastWriteTime -le (Get-Date "2023-06-22"))} | Select-Object FullName, CreationTime, LastWriteTime

```

- **Search for File by Hash**:  

```powershell

Get-ChildItem -Path C:\ -Recurse -File | Get-FileHash | Where-Object Hash -eq 'HASH_VALUE' | Select-Object Path

```

- **Search for File by Name**:  

```powershell

Get-ChildItem -Path "C:\Path\To\Search" -Filter "FileName"

```

- **File Changed in Last 24 Hours**:  

```powershell

Get-ChildItem -Path C:\ -Recurse -File -Force | Where-Object { $_.LastWriteTime -ge (Get-Date).AddHours(-6) }

```

- **Files Created in Last 24 Hours**:  

```powershell

Get-ChildItem -Path C:\ -Recurse -File -Force | Where-Object { $_.CreationTime -ge (Get-Date).AddHours(-6) }

```

- **Obtain List of All Files**:  

```powershell

tree C:\ /F > output.txt

dir C:\ /A:H /-C /Q /R /S /X

```

- **Delete Registry Key**:  

```powershell

Remove-Item -Path "HKLM:\Path\To\Registry\Key" -Recurse

```

- **Read Registry Key**:  

```powershell

Get-ChildItem -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\'

```

  
- **Retrieve Auto-Start Locations (Startup Programs)**
```powershell
Get-WmiObject -Query "SELECT * FROM Win32_StartupCommand" | Select-Object Name, Command, User

```

- **List Installed Programs**
```powershell
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallDate

```
## Event Logs

- **Search Sysmon Logs**:  

```powershell

Get-WinEvent -FilterHashtable @{logname="Microsoft-Windows-Sysmon/Operational"; id=1} | Where-Object {$_.Properties[20].Value -like "*rdpclip*"} | fl

```

- **Login History**:  

```powershell

Get-WinEvent -FilterHashtable @{LogName='Security';ID=4624,4625} | Select-Object TimeCreated, Id, Message

```

- **Search Event Logs by Keyword**:  

```powershell

Get-WinEvent -FilterHashtable @{ LogName='Security';} | Select Timecreated,LogName,Message | Where {$_.Message -like "*test*"} | FL

```

- **Convert EVTX to CSV**:  

```powershell

Get-WinEvent -Path .\Microsoft-Windows-Sysmon%4Operational.evtx | Export-CSV foo.csv

```

  

## User and Account Information

- **Local Users Information**:  

```powershell

Get-LocalUser | Select *

```

- **Get the SID from a Given Username**:  

```powershell

$username = "Domain\Username"

$account = New-Object System.Security.Principal.NTAccount($username)

$sid = $account.Translate([System.Security.Principal.SecurityIdentifier]).Value

Write-Output $sid

```

- **Translate SIDs**:  

```powershell

$SID = "S-1-5-21-329068152-1454471165-1417001333-12984448"

$account = New-Object System.Security.Principal.SecurityIdentifier($SID)

$username = $account.Translate([System.Security.Principal.NTAccount]).Value

Write-Output $username

```

  

## Job and Task Management

- **Scheduled Tasks**:  

```powershell

Get-ScheduledTask | Where-Object { $_.State -eq 'Ready' }

```


```powershell

Get-ScheduledJob

```

  

## File Operations

- **Copy File**:  

```powershell

Copy-Item -Source \\server\share\test -Destination C:\path\

```

- **Download File from Internet**:  

```powershell

Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile $env:TEMP\sysmon.zip

```

- **Unzip File**:  

```powershell

Expand-Archive -Path <SourcePathofZipFile> -DestinationPath <DestinationPath>

```

- **Encrypt File**:  

```powershell

Read-Host -Prompt "Enter the encryption passphrase" -AsSecureString | ConvertFrom-SecureString | Set-Content -Path "C:\Path\EncryptedFile.txt"

```

  

## Web Operations

- **Upload File to Web**:  

```powershell

$filePath = "C:\Path\To\File.txt"

$uploadUrl = "http://example.com/upload"

Invoke-RestMethod -Uri $uploadUrl -Method POST -InFile $filePath

```

  

## Miscellaneous

- **Generate Random Data (10MB)**:  

```powershell

$fileSizeInMB = 10; $filePath = "$PWD\file.txt"; $data = [byte[]]::new($fileSizeInMB * 1MB); $random = New-Object -TypeName System.Random; $random.NextBytes($data); [System.IO.File]::WriteAllBytes($filePath, $data)

```

- **Regex Matching for Specific String ("feras")**:  

```powershell

Select-String -Path C:\path\strings.txt -Pattern '\bferas\b' -AllMatches | % { $_.Matches } | % { $_.Value }

```


**Linux cut command**

```powershell
Cut function powershell

    function cut {
    param(
        [Parameter(ValueFromPipeline=$True)] [string]$inputobject,
        [string]$delimiter='\s+',
        [string[]]$field
    )

    process {
        if ($field -eq $null) { $inputobject -split $delimiter } else {
        ($inputobject -split $delimiter)[$field] }
    }
    }
```


---
## PowerShell with Security Utilities

PowerShell enhances security utilities like Sysinternals for effective IR:

- **Sysinternals**: Start utilities like Process Explorer:
  ```powershell
  Start-Process "C:\Path\To\ProcessExplorer.exe"
  ```

- **Remote Commands**: Use PsExec to run commands on target machines:
  ```powershell
  .\PsExec.exe \\TargetMachine cmd
  ```

- **Data Collection**: Retrieve logs from Sysmon:
  ```powershell
  Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational'
  ```

 
---
 


