---
layout: post
date: 30/09/2022
author: Aswin Gopalakrishnan
title: Enumeration Methods Overview
---

To gather data within an Active Directory (AD) environment, various methods and tools are available:

- **ADSI (Active Directory Service Interfaces):** A set of COM interfaces used for accessing the capabilities of directory services from different network providers in a distributed computing environment.
- **.NET Classes:** .NET Framework classes that enable directory services management and interaction with AD.
- **Native Executable:** Executables that can be directly run on the command line or PowerShell for AD querying.
- **WMI (Windows Management Instrumentation) using PowerShell:** A powerful scripting language that can manage and query devices.
- **ActiveDirectory Module:** PowerShell module specifically for AD management, providing a comprehensive set of cmdlets.
- **Automated Tools (GUI):** These include:
  - **BloodHound:** Graphical AD recon tool for understanding relationships and paths to target high-privilege accounts.
  - **pingcastle:** AD security scanning tool focusing on risk assessments.
  - **purple knight:** Security assessment tool that identifies vulnerabilities in AD configurations.


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
|Get-ForestDomain -Verbose \| Get-DomainTrust \| ?{$\_.TrustAttributes -eq 'FILTER_SIDS'}|List External Trust of the current forest.|PowerView|
|(Get-ADForest).domains \| % {Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)' -Server $\_ } |List External Trust of the current forest| Active Directory |
|Get-ADTrust -Filter 'intraForest -ne $True' -Server (Get-ADForest).Name|List all external forest trusts | Active Directory |  
|Get-ADTrust -Filter * -Server eu.local |Enumerate forest trusts of a remote forest | Active Directory|
|Get-DomainTrust -Domain techcorp.local \| ?{ $\_.TrustAttributes -match 'FOREST_TRANSITIVE' } |List domain external trusts | PowerView|


## Domain
Active Directory Domain is a centralized database of user accounts and computer information that enables administrators to manage and control access to network resources. It provides a hierarchical structure that allows for the organization and management of network resources, such as users, computers, and servers. In simpler terms, it is a way to manage and organize access to resources in a network.


| Command 											  | 		Description 					|   Module 	       |
| :---     											  |   		:---:     						|    	   ---:    |
| (Get-ADDomain).Get-DomainSID						  |  Find Domain SID 						|Active Directory|
| Get-ADDomainController -DomainName moneycorp.local  |  Find Domain Controller 				|Active Directory|
| Get-DomainController 								  |  Find Domain Controller 				|PowerView|
| Get-DomainController –Domain moneycorp.local        |  Find Domain Controller of a domain 	|PowerView|
| Get-DomainSID 									  |  Find Domain SID 						|PowerView|
|(Get-DomainPolicy)."system access"				      |  Enumerate Domain Policy 				|PowerView|
|(Get-DomainPolicy)."KerberosPolicy"                  |  Enumerate Kerberos Policy 			    |PowerView|
|(Get-DomainPolicy –domain moneycorp.local)."KerberosPolicy"| Enumeate Kerberos Policy of a domain|PowerView|


## Users
Active Directory user objects are containers for storing and managing information about users in a network environment. They typically include attributes such as the user's name, username, password, email address, and group memberships. User objects allow administrators to control access to network resources, assign permissions, and manage user settings across the network.

| Command 											  | 		Description 					|   Module 	       |
| :---     											  |   		:---:     						|    	   ---:    |
|Get-DomainUser –Username student1 					  |	Find Last Set Password   				| 	 PowerView 	   |
|Get-UserProperty –Properties pwdlastset			  |	Search Desription for keyword "built"   | 	 PowerView 	   |
|Find-UserField -SearchField Description -SearchTerm "built"|  Enumerate User Properties 		| 	 PowerView 	   |
|Get-DomainUser -LDAPFilter "Description=*built*" \|  Select name,Description|Enumerate User Properties | 	 PowerView 	   |

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
```bash
Get-ADGroup -Filter * -Properties *
Get-NetGroup –Domain <targetdomain>
```

Enumerate groups with name contain "admin" keyword and list only names.
```bash
Get-ADGroup -Filter 'Name -like "*admin*"' | select Name
```

Enumerate the group members of **Domain Admins** group.
```bash
Get-ADGroupMember -Identity "Domain Admins" -Recursive
Get-DomainGroupMember -Identity "studentusers" -Recurse
```

Find all groups the current user is part of.
```bash
Get-ADGroup -Filter * | % { Write-Host "The Group Name is '$_.Name'" -ForegroundColor red;Get-ADGroupMember -Identity $_.SamAccountName | select Name,ObjectClass }

Get-DomainGroup -UserName studentuser19 | select name
```

Find Local Group in a computer of **Users** Group.
```bash
Get-LocalGroupMember -Group "Users"
```

List all local groups.
```bash
Get-LocalGroup
```

## LoggedOn

Find all logged in users on the domain joined computer.
```bash
Get-NetLoggedon –ComputerName <servername>
```

Retrieve information about users who are currently logged on to a local computer. 
```bash
Get-LoggedonLocal -ComputerName dcorp-dc.dollarcorp.moneycorp.local
```

Retrieve information about the user last logged on the domain joined computer.
```bash
Get-LastLoggedOn –ComputerName <servername>
```

## Group Policy Object (GPO)

List all group policies.
```bash
Get-DomainGPO
```

List domain GPO assigned to a specific computer.
```bash
Get-DomainGPO -ComputerIdentity student19.us.techcorp.local
```

List domain GPO assigned to all the computer of the current domain.
```bash
Get-DomainComputer | %{ Write-host $_.dnshostname -ForegroundColor red; Get-DomainGPO -ComputerIdentity $_.dnshostname | select displayname; Write-host "============================" -ForegroundColor red} 2> $null
```

Finds users who have local admin rights over WINDOWS3 through GPO correlation.
```bash
Get-DomainGPOComputerLocalGroupMapping -ComputerName WINDOWS3.testlab.local
```

Finds information about how the "RDP" local group on the "WINDOWS4.dev.testlab.local" computer is affected by GPOs in the "dev.testlab.local" domain.
```bash
Get-DomainGPOComputerLocalGroupMapping -Domain dev.testlab.local -ComputerName WINDOWS4.dev.testlab.local -LocalGroup RDP
```

Find all users who have local admin rights across the machines in the network.
```bash
Get-DomainComputer -domain glacis.corp | % { Get-DomainGPOComputerLocalGroupMapping -ComputerName $_.dnshostname -domain glacis.corp | Format-Table -Property ObjectName, GPODisplayName, IsGroup, ComputerName, GPOType -AutoSize } 2>$null
```

Find all user/group -> machine relationships where the user/group is a member of the local administrators group on target machines.
```bash
Get-DomainUser | % { Get-DomainGPOUserLocalGroupMapping -Identity $_.samaccountname | Format-Table -Property ObjectName, GPODisplayName,IsGroup, ObjectName, GPOType, ComputerName } 2> $null
```

Returns all GPOs in a domain that modify local group memberships through 'Restricted Groups'.
```bash
Get-NetGPOGroup -ResolveMembersToSIDs
```

Get users which are in a local group of a machine using GPO.
```bash
Find-GPOComputerAdmin –Computername dcorp-student1.dollarcorp.moneycorp.local
```

Get users which are in a local group in all machines of the domain using GPO.
```bash
Get-DomainComputer | % { Write-Output $_.dnshostname;Find-GPOComputerAdmin -ComputerName $_.dnshostname }
```

Get machines where the given user is member of a specific group.
```bash
Get-DomainUser -domain glacis.corp | % { Write-Output $_.name;Find-GPOLocation -Identity $_.name 2> $null }
```

## Organisational Unit (OU)
Find all OUs and the GPO Applied.
```bash
((Get-DomainOU)."gplink").substring(11,38) | %{ Write-Host "The OU is $((Get-DomainOU -GPLink $_)."displayname")" -ForegroundColor red; Get-DomainGPO -Identity $_ | Format-Table -Property displayname -AutoSize}
```

Get users which are in a local group of a machine in any OU using GPO.
```bash
(Get-DomainOU -domain glacis.corp).distinguishedname | %{Get-DomainComputer -SearchBase $_} | Get-DomainGPOComputerLocalGroupMapping 2> $null
```

Get users which are in a local group of a machine in a particular OU using GPO
```bash
(Get-DomainOU -Identity 'OU=Mgmt,DC=us,DC=techcorp,DC=local').distinguishedname | %{Get- DomainComputer -SearchBase $_} | Get-DomainGPOComputerLocalGroupMapping
```

Find all computers in the Student OU.
```bash
(Get-DomainOU -domain glacis.corp -Identity 'OU=Servers,DC=citadel,DC=corp').distinguishedname | % { Get-DomainComputer -SearchBase $_ } | select dnshostname
```

Find all OU, machine in them and GPOs attached.
```bash
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
```bash
(Get-Acl 'AD:\CN=Administrator,CN=Users,DC=us,DC=techcorp,DC=local').Access
```

Get the ACLs associated with the specified path
```bash
Get-PathAcl -Path "\\us-dc\sysvol"
```

Get the ACLs associated with the specified domain object.
```bash
Get-DomainObjectAcl -Searchbase "LDAP://CN=Domain Admins,CN=Users,DC=us,DC=techcorp,DC=local" -ResolveGUIDs -Verbose
Get-ObjectAcl -ADSpath "LDAP://CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs - Verbose
```

ACL of the user/group on other objects
```bash
Find-InterestingDomainAcl | ?{$_.identityreferencename -match 'webmaster' -or $_.identityreferencename -match 'serviceaccount' } | select identityreferencename,ActiveDirectoryRights,ObjectDN,AceQualifier,identityreferenceClass
```

ACL applied on the object
```bash
Find-InterestingDomainAcl -ResolveGUIDs | ? { $_.ObjectDN -like '*MachineAdmins*'} | select IdentityReferenceName,ActiveDirectoryRights,ObjectDN | Format-Table -AutoSize -HideTableHeaders -Wrap
```

## Trust Mapping
Get a list of all domain trusts for the current domain
```bash
Get-NetDomainTrust –Domain us.dollarcorp.moneycorp.local
```

Map trusts of a forest
```bash
Get-NetForestTrust –Forest eurocorp.local 
Get-ADTrust -Filter 'msDS-TrustForestTrustInfo -ne "$null"'
```

## User Hunting
Find all machines on the current domain where the current user has local admin access.
```bash
Find-LocalAdminAccess –Verbose
```

This can also be done with the help of remote administration tools like WMI.
```bash
Find-WMILocalAdminAccess.ps1
```

Find all machine with local admin access with PS remoting.
```bash
Find-PSRemotingLocalAdminAccess.ps1
```

Find local admins on all machines of the domain (needs administrator privs on non-dc machines.
```bash
Invoke-EnumerateLocalAdmin
```

Track the movements of specific user accounts within a Windows domain.
```bash
Find-DomainUserLocation -Verbose
Find-DomainUserLocation -UserGroupIdentity "StudentUsers"
Find-DomainUserLocation -CheckAccess
Find-DomainUserLocation –Stealth
```

Find user that belong to a specified Active Directory group.
```bash
Invoke-UserHunter -GroupName "RDPUsers"
```

Find user that belong to a specified Active Directory group has access on which computer.
```bash
Invoke-UserHunter -CheckAccess
```

When the -Stealth parameter is used, the command attempts to perform its operations in a way that is less likely to be detected by security monitoring tools or alert administrators. 
```bash
Invoke-UserHunter -Stealth
```

## File Share
Find shares on hosts in current domain.
```bash
Invoke-ShareFinder –Verbose
```

Find sensitive files on computers in the domain
```bash
Invoke-FileFinder –Verbose
```

Get all fileservers of the domain
```bash
Get-NetFileServer
```

## Forest Mapping
Mapping forests provides insight into domains, catalogs, and structural hierarchies within the forest.
```bash
Get-Forest # Returns details on the forest, including trust relationships.
Get-ForestDomain # Lists all domains within the forest.
```

## MSSQL Servers
Identifies SQL server instances and checks accessibility.

### SPN Scanning
- ```bash 
Get-SQLInstanceDomain # Finds SQL instances within the domain by scanning SPNs.
```

### Accessibility Check
- ```bash 
Get-SQLConnectionTestThreaded # Tests SQL server accessibility for the current user.
```

### Information Gathering
- ```bash 
Get-SQLServerInfo # Gathers details on SQL servers, such as accessible accounts and configurations.
```

---
