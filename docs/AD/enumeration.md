---
layout: default
---

# Enumeration
During Active Directory enumeration, a red teamer will typically use various tools and techniques to collect information about the target environment, such as the domain structure, users and groups, computer systems, and network resources. This information can be used to identify potential attack vectors, privilege escalation opportunities, and other weaknesses that can be exploited to gain unauthorized access to the target environment.

By understanding and effectively utilizing these commands, red teamers can more efficiently navigate Active Directory and achieve their objectives. However, it's important to use these commands ethically and with proper authorization.


## Forest
Active Directory Forests are a hierarchical way to manage multiple domains sharing a common schema and configuration. It provides a flexible and scalable approach to managing resources within an organization.

| Command 					| 		Description 					|   Module 	       |
| :--     					|   		:---:     					|    	   ---:    |
|Get-ADForest				|Get domains from the current forest.   | Active Directory |
|Get-Forest 				|Map trusts of a forest     			| PowerView 	   |
|Get-ForestDomain 			|Map domains in a forest 				| Active Directory |
|(Get-ADForest).Domains 	|Map domains in a forest 				| Active Directory |  
|Get-ForestGlobalCatalog    |Get Global Catalogue 					| PowerView		   |


## Trusts
Active Directory trusts allow users in one Windows Server domain to access resources in another domain by establishing a trust relationship between them.


| Command 					| 		Description 					|   Module 	       |
| :--     					|   		:---:     					|    	   ---:    |
|Get-ForestTrust			|Map trusts of a forest.  		 		| Active Directory |
|Get-ForestDomain -Verbose \| Get-DomainTrust | ?{$\_.TrustAttributes -eq 'FILTER_SIDS'}|List External Trust of the current forest.| PowerView|
|(Get-ADForest).domains \| % {Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)' -Server $\_ } |List External Trust of the current forest| Active Directory |
|Get-ADTrust -Filter 'intraForest -ne $True' -Server (Get-ADForest).Name|List all external forest trusts | Active Directory |  
|Get-ADTrust -Filter * -Server eu.local |List all external forest trusts | Active Directory|
|Get-DomainTrust -Domain techcorp.local \| ?{ $\_.TrustAttributes -match 'FOREST_TRANSITIVE' } |List domain external trusts | Active Directory|


## Domain

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

## Users
Enumerate domain users.
```
Get-NetUser –Username student1
Get-UserProperty –Properties pwdlastset
Find-UserField -SearchField Description -SearchTerm "built"
Get-DomainUser -LDAPFilter "Description=*built*" |   Select name,Description
```

## Computers
Enumerate Computers
```
Get-ADComputer -Filter 'OperatingSystem -like "*Server 2016*"' - Properties OperatingSystem | select Name,OperatingSystem
```

Ping all machines in the current domain
```
Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName}
```


## Groups
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

Get-DomainGroup -UserName studentuser19 | select name
```

Find Local Group in a computer of **Users** Group.
```
Get-LocalGroupMember -Group "Users"
```

List all local groups.
```
Get-LocalGroup
```

## LoggedOn

Find all logged in users on the domain joined computer.
```
Get-NetLoggedon –ComputerName <servername>
```

Retrieve information about users who are currently logged on to a local computer. 
```
Get-LoggedonLocal -ComputerName dcorp-dc.dollarcorp.moneycorp.local
```

Retrieve information about the user last logged on the domain joined computer.
```
Get-LastLoggedOn –ComputerName <servername>
```

## Group Policy Object (GPO)

List all group policies.
```
Get-DomainGPO
```

List domain GPO assigned to a specific computer.
```
Get-DomainGPO -ComputerIdentity student19.us.techcorp.local
```

List domain GPO assigned to all the computer of the current domain.
```
Get-DomainComputer | %{ Write-host $_.dnshostname -ForegroundColor red; Get-DomainGPO -ComputerIdentity $_.dnshostname | select displayname; Write-host "============================" -ForegroundColor red} 2> $null
```

Finds users who have local admin rights over WINDOWS3 through GPO correlation.
```
Get-DomainGPOComputerLocalGroupMapping -ComputerName WINDOWS3.testlab.local
```

Finds information about how the "RDP" local group on the "WINDOWS4.dev.testlab.local" computer is affected by GPOs in the "dev.testlab.local" domain.
```
Get-DomainGPOComputerLocalGroupMapping -Domain dev.testlab.local -ComputerName WINDOWS4.dev.testlab.local -LocalGroup RDP
```

Find all users who have local admin rights across the machines in the network.
```
Get-DomainComputer -domain glacis.corp | % { Get-DomainGPOComputerLocalGroupMapping -ComputerName $_.dnshostname -domain glacis.corp | Format-Table -Property ObjectName, GPODisplayName, IsGroup, ComputerName, GPOType -AutoSize } 2>$null
```

Find all user/group -> machine relationships where the user/group is a member of the local administrators group on target machines.
```
Get-DomainUser | % { Get-DomainGPOUserLocalGroupMapping -Identity $_.samaccountname | Format-Table -Property ObjectName, GPODisplayName,IsGroup, ObjectName, GPOType, ComputerName } 2> $null
```

Returns all GPOs in a domain that modify local group memberships through 'Restricted Groups'.
```
Get-NetGPOGroup -ResolveMembersToSIDs
```

Get users which are in a local group of a machine using GPO.
```
Find-GPOComputerAdmin –Computername dcorp-student1.dollarcorp.moneycorp.local
```

Get users which are in a local group in all machines of the domain using GPO.
```
Get-DomainComputer | % { Write-Output $_.dnshostname;Find-GPOComputerAdmin -ComputerName $_.dnshostname }
```

Get machines where the given user is member of a specific group.
```
Get-DomainUser -domain glacis.corp | % { Write-Output $_.name;Find-GPOLocation -Identity $_.name 2> $null }
```

## Organisational Unit (OU)
Find all OUs and the GPO Applied.
```
((Get-DomainOU)."gplink").substring(11,38) | %{ Write-Host "The OU is $((Get-DomainOU -GPLink $_)."displayname")" -ForegroundColor red; Get-DomainGPO -Identity $_ | Format-Table -Property displayname -AutoSize}
```

Get users which are in a local group of a machine in any OU using GPO.
```
(Get-DomainOU -domain glacis.corp).distinguishedname | %{Get-DomainComputer -SearchBase $_} | Get-DomainGPOComputerLocalGroupMapping 2> $null
```

Get users which are in a local group of a machine in a particular OU using GPO
```
(Get-DomainOU -Identity 'OU=Mgmt,DC=us,DC=techcorp,DC=local').distinguishedname | %{Get- DomainComputer -SearchBase $_} | Get-DomainGPOComputerLocalGroupMapping
```

Find all computers in the Student OU.
```
(Get-DomainOU -domain glacis.corp -Identity 'OU=Servers,DC=citadel,DC=corp').distinguishedname | % { Get-DomainComputer -SearchBase $_ } | select dnshostname
```

Find all OU, machine in them and GPOs attached.
```
foreach($name in $(Get-DomainOU | select -ExpandProperty distinguishedname)){
Write-Host "========== $name ============" -ForegroundColor red;
$computer = Get-DomainComputer -SearchBase $name | select -ExpandProperty dnshostname 2>$null;
foreach($comp in $computer){
$gpo = Get-DomainGPO -ComputerIdentity $comp 2>$null | select -ExpandProperty displayname;
Write-Host "$comp -> $gpo" 2>$null;
}};
```

## ACL
Get the ACLs associated with the specified user
```
(Get-Acl 'AD:\CN=Administrator,CN=Users,DC=us,DC=techcorp,DC=local').Access
```

Get the ACLs associated with the specified path
```
Get-PathAcl -Path "\\us-dc\sysvol"
```

Get the ACLs associated with the specified domain object.
```
Get-DomainObjectAcl -Searchbase "LDAP://CN=Domain Admins,CN=Users,DC=us,DC=techcorp,DC=local" -ResolveGUIDs -Verbose
Get-ObjectAcl -ADSpath "LDAP://CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs - Verbose
```

ACL of the user/group on other objects
```
Find-InterestingDomainAcl | ?{$_.identityreferencename -match 'webmaster' -or $_.identityreferencename -match 'serviceaccount' } | select identityreferencename,ActiveDirectoryRights,ObjectDN,AceQualifier,identityreferenceClass
```

ACL applied on the object
```
Find-InterestingDomainAcl -ResolveGUIDs | ? { $_.ObjectDN -like '*MachineAdmins*'} | select IdentityReferenceName,ActiveDirectoryRights,ObjectDN | Format-Table -AutoSize -HideTableHeaders -Wrap
```

## Trust Mapping
Get a list of all domain trusts for the current domain
```
Get-NetDomainTrust –Domain us.dollarcorp.moneycorp.local
```

Map trusts of a forest
```
Get-NetForestTrust –Forest eurocorp.local 
Get-ADTrust -Filter 'msDS-TrustForestTrustInfo -ne "$null"'
```

## User Hunting
Find all machines on the current domain where the current user has local admin access.
```
Find-LocalAdminAccess –Verbose
```

This can also be done with the help of remote administration tools like WMI.
```
Find-WMILocalAdminAccess.ps1
```

Find all machine with local admin access with PS remoting.
```
Find-PSRemotingLocalAdminAccess.ps1
```

Find local admins on all machines of the domain (needs administrator privs on non-dc machines.
```
Invoke-EnumerateLocalAdmin
```

Track the movements of specific user accounts within a Windows domain.
```
Find-DomainUserLocation -Verbose
Find-DomainUserLocation -UserGroupIdentity "StudentUsers"
Find-DomainUserLocation -CheckAccess
Find-DomainUserLocation –Stealth
```

Find user that belong to a specified Active Directory group.
```
Invoke-UserHunter -GroupName "RDPUsers"
```

Find user that belong to a specified Active Directory group has access on which computer.
```
Invoke-UserHunter -CheckAccess
```

When the -Stealth parameter is used, the command attempts to perform its operations in a way that is less likely to be detected by security monitoring tools or alert administrators. 
```
Invoke-UserHunter -Stealth
```

## File Share
Find shares on hosts in current domain.
```
Invoke-ShareFinder –Verbose
```

Find sensitive files on computers in the domain
```
Invoke-FileFinder –Verbose
```

Get all fileservers of the domain
```
Get-NetFileServer
```

## Forest Mapping
Get all domains in the current forest
```
Get-NetForestDomain –Forest eurocorp.local
(Get-ADForest).Domains
Get-NetForestCatalog
```

## MSSQL Servers
Find MSSQL Instance ( Via SPN Scanning)
```
Get-SQLInstanceDomain
```

Check whether the current user has access on the server.
```
Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded -Verbose
```

Gather information. Check which service account may access the server.
```
Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose
```