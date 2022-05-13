---
layout: page
title: Wintel Alphabook
tags: 
 - network
 - windows
 - active directory
permalink: /
---

# Welcome to Wintel Alphabook


![assets/img/ad.png](assets/img/ad.png)

## Network Topology

Network is the basis of the whole IT infrastructure. We can not imagine the situation without network.

IP Addresses reserved for use on private networks

{% include alert.html type="info" title="Class A" content="10.0.0.0 to 10.255.255.255" %}

{% include alert.html type="primary" title="Class B" content="172.16.0.0 to 172.31.255.255" %}

{% include alert.html type="secondary" title="Class C" content="192.168.0.0 to 192.168.255.255" %}

### Choosing private IPv4 address

1. We need to reserve enough IP address for all devices (Desktop, Laptop, Mobile, etc), Class A may be good choose. 

2. Pay attention to potential IP confiction in your environment (Docker may use IP address of Class B, etc).

### Example of a company with three Offices

```bash
Beijing Office 10.10.0.0/255.255.0.0
Shanghai Office 10.20.0.0/255.255.0.0
Shenzhen Office 10.30.0.0/255.255.0.0
```

### Example of Beijing office

```bash
Windows Server 10.10.10.0/255.255.255.0 (Available IP address 10.10.10.1 - 10.10.10.254)
Linux Server 10.10.20.0/255.255.254.0 (Available IP address 10.10.20.1 - 10.10.21.254)
```

Windows Domain Controller
```bash
BJDC01 10.10.10.10/255.255.255.0
BJDC02 10.10.10.11/255.255.255.0
```

## Active Directory Domain Services Design

* Single forest single domain is preferred, 2,150,000,000 objects per domain, FQDN less than 64 characters
* Selecting the Forest Root Domain (corp.alphabook.cn) https://technet.microsoft.com/en-us/library/cc726016(v=ws.10).aspx
* PDC acts as a key role (time root in forest, etc)
* Implement multiple/backup domain controllers (all GCs)
* Implement multiple sites as needed (site, subnet,site link(180 minutes default, 15 minutes minimum), bridge all site links(default))
* Enable [[Active Directory Recycle Bin]]
* Enable [[Protect object from accidental deletion]] (User, OU, etc)
* Fill more AD account properties (City, Country, Phone, Job Title, Department, Manager...) [[Create AD user with PowerShell]]
* Grant permission per group, not a single user
* [[Account Lockout]] Policy
* [[Delegate IT helpdesk group join computer into domain permission]]

## Install first domain controller

### Install Windows Server 2019

```bash
# Check Operating System version
(Get-WmiObject -Class Win32_OperatingSystem).Caption
Microsoft Windows Server 2019 Standard
```

### Rename hostname

```bash
# BJ stands for Beijing
# DC stands for Domain Controller
Rename-Computer BJDC01
Restart-Computer
Get-Content Env:COMPUTERNAME
```

### Configure IP information

```bash
IP address: 10.10.10.10
Subnet mask: 255.255.255.0
Default gateway: 10.10.10.1
Preferred DNS server: 10.10.10.10
```

### Install Active Directory Domain Services

```bash
From Start Menu, Click "Server Manager"
Click "Add roles and features"
Click "Next"
Keep default option "Role-based or feature-based installation", Click "Next"
Keep defaut option, Click "Next"
Select "Active Directory Domain Services", and Click "Add Features" from the pop up window
Click "Next"
Click "Next"
Click "Next"
Click "Install"
Click "Close" when finish
```

### Post-deployment Configuration

![assets/img/dc-promote.png](assets/img/dc-promote.png)

```bash
Click "Flag" icon
Click "Promote this server to a domain controller"
Select "Add a new forest", input "alphabook.cn" as "Root domain name", click "Next"
Keep default option
# Forest functional level: Windows Server 2016
# Domain functional level: Windows Server 2016
# Domain Name System (DNS) Server checked
# Global Catalog (GC) checked
# Read only domain controller (RODC) unchecked and unavailabled for the first Domain Controller
Input the Directory Services Restore Mode (DSRM) password, click "Next"
Keep default DNS Options, click "Next"
Keep default NetBIOS domain name, click "Next"
Keep default options of AD DS database, log files, and SYSVOL, click "Next"
Click "Next"
Click "Install" after Prerequisites Check done
```

### Configure time

```bash
# By default, the first domain controller is PDC too, PDC is the time root of the forest.
w32tm /config /computer:BJDC01.alphabook.cn /manualpeerlist:time.windows.com /syncfromflags:manual /update
```

### FSMO Role Holders

{% include alert.html type="warning" title="Schema master / Forest level" content="To make change Schema in forest (such as implement Exchange, Lync, SCCM)" %}

{% include alert.html type="danger" title="Domain naming master / Forest level" content="To add/remove domain in forest" %}

{% include alert.html type="success" title="PDC / Domain level" content="Time root in forest (PDC->DCs->Computers)
Group policy central management
Handle password change specially (the change will sync to PDC immediately)
Handle user account lock specially" %}

{% include alert.html type="info" title="RID pool master / Domain level" content="Assign RIDs to DCs (500/time)" %}

{% include alert.html type="primary" title="Infrastucture master / Domain level" content="Objects reference in different domains" %}

Get list of FSMO role holders
```bash
netdom query fsmo
Schema master               BJDC01.alphabook.cn
Domain naming master        BJDC01.alphabook.cn
PDC                         BJDC01.alphabook.cn
RID pool manager            BJDC01.alphabook.cn
Infrastructure master       BJDC01.alphabook.cn
```

or
```bash
# Forest Level
Get-ADForest | Select-Object DomainNamingMaster, SchemaMaster
DomainNamingMaster  SchemaMaster       
------------------  ------------       
BJDC01.alphabook.cn BJDC01.alphabook.cn
# Domain Level
Get-ADDomain | Select-Object InfrastructureMaster, RIDMaster, PDCEmulator
InfrastructureMaster RIDMaster           PDCEmulator        
-------------------- ---------           -----------        
BJDC01.alphabook.cn  BJDC01.alphabook.cn BJDC01.alphabook.cn
```


## Install second/backup domain controller
