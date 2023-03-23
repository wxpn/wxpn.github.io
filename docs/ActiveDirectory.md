---
layout: default
---

# Active Directory

Active Directory is a technology used in Windows environments that allows administrators to manage users, computers, and other network resources. As a red teamer, understanding how to interact with and manipulate Active Directory can be crucial in achieving your objectives. Knowing the right commands to use can save time and increase the effectiveness of your operations.

By understanding and effectively utilizing these commands, red teamers can more efficiently navigate Active Directory and achieve their objectives. However, it's important to use these commands ethically and with proper authorization.


## Enumeration

### Forest
Get Current Forest.
```
Get-ADForest
Get-Forest
```

Get domains from the current forest
```
(Get-ADForest).Domains
Get-ForestDomain
```

Get all global catalogs for the current forest
```
Get-ForestGlobalCatalog`
```

Map trusts of a forest
```
Get-ForestTrust
```

### Trusts
List External Trust of the current forest.
```
Get-ForestDomain -Verbose | Get-DomainTrust | ?{$_.TrustAttributes -eq 'FILTER_SIDS'}
(Get-ADForest).domains | % { Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)' -Server $_ }
```

List all external forest trusts
```
Get-ADTrust -Filter 'intraForest -ne $True' -Server (Get- ADForest).Name
Get-ADTrust -Filter * -Server eu.local
Get-DomainTrust -Domain techcorp.local | ?{ $_.TrustAttributes -match 'FOREST_TRANSITIVE' }
```

### Domain

Using AD Module:

```
Get-ADDomain
Get-ADDomain -Identity moneycorp.local
(Get-ADDomain).DomainSID
Get-ADDomainController -DomainName moneycorp.local
```

Using PowerView module:

```
Get-DomainSID
(Get-DomainPolicy)."system access"
(Get-DomainPolicy)."KerberosPolicy"
(Get-DomainPolicy –domain moneycorp.local)."KerberosPolicy"
Get-NetDomain
Get-NetDomain –Domain moneycorp.local
Get-NetDomainController
Get-NetDomainController –Domain moneycorp.local
```

### Users
Enumerate domain users.
```
Get-NetUser –Username student1
Get-UserProperty –Properties pwdlastset
Find-UserField -SearchField Description -SearchTerm "built"
Get-DomainUser -LDAPFilter "Description=*built*" |   Select name,Description
```

### Computers
Enumerate Computers
```
Get-ADComputer -Filter 'OperatingSystem -like "*Server 2016*"' - Properties OperatingSystem | select Name,OperatingSystem
```

Ping all machines in the current domain
```
Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName}
```


### Groups
Enumerate all groups of a current domain
```
Get-ADGroup -Filter * -Properties *
Get-NetGroup –Domain <targetdomain>
```
Enumerate groups with name contain "admin" keyword and list only names.
```
Get-ADGroup -Filter 'Name -like "*admin*"' | select Name
```
Enumerate the group members of **Domain Admins** group.
```
Get-ADGroupMember -Identity "Domain Admins" -Recursive
Get-DomainGroupMember -Identity "studentusers" -Recurse
```
Find all groups the current user is part of.
```
Get-ADGroup -Filter * | % { Write-Host "The Group Name is '$_.Name'" -ForegroundColor red;Get-ADGroupMember -Identity $_.SamAccountName | select Name,ObjectClass }

Get-DomainGroup -UserName studentuser19

```
Find Local Group in a computer of **Users** Group.
```
Get-LocalGroupMember -Group "Users"
```
List all local groups.
```
Get-LocalGroup
```

### LoggedOn

Find all logged in users on the domain joined computer.
```
Get-NetLoggedon –ComputerName <servername>
```

Retrieve information about users who are currently logged on to a local computer. 
```
Get-LoggedonLocal -ComputerName dcorp- dc.dollarcorp.moneycorp.local
```

Retrieve information about the user last logged on the domain joined computer.
```
Get-LastLoggedOn –ComputerName <servername>
```

[back](./)
