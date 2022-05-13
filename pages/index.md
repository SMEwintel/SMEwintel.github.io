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

## Windows Active Directory

### Active Directory Domain Service Design

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

### Install first domain controller

1. Install Windows Server 2019 Standard Operation System
```bash
(Get-WmiObject -Class Win32_OperatingSystem).Caption
Microsoft Windows Server 2019 Standard
```

2. Rename hostname
```bash
# BJ stands for Beijing
# DC stands for Domain Controller
Rename-Computer BJDC01
Restart-Computer
Get-Content Env:COMPUTERNAME
```

3. Configure IP information as below:
```bash
IP address: 10.10.10.10
Subnet mask: 255.255.255.0
Default gateway: 10.10.10.1
Preferred DNS server: 10.10.10.10
```


### Install second/backup domain controller
