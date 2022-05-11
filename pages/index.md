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

![assets/img/ab.png](assets/img/ab.png)

## Network Topology

Network is the basis of the whole IT infrastructure. We can not imagine the situation without network.

IP Addresses reserved for use on private networks

{% include alert.html type="info" title="Class A" content="10.0.0.0 to 10.255.255.255" %}

{% include alert.html type="primary" title="Class B" content="172.16.0.0 to 172.31.255.255" %}

{% include alert.html type="secondary" title="Class C" content="192.168.0.0 to 192.168.255.255" %}

### Choosing private IPv4 address

1. We need to reserve enough IP address for all devices (Desktop, Laptop, Mobile, etc), Class A may be good choose. 

2. Pay attention to potential IP confiction in your environment (Docker may use IP address of Class B, etc).

## Windows Active Directory

