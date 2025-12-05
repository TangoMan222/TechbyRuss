---
layout: default
title: "Segmenting my homelab with VLANs and UniFi"
date: 2025-12-02
tags: ["networking", "homelab", "unifi"]
---

# Segmenting my homelab with VLANs and UniFi

I have been rebuilding my homelab to better separate management traffic, lab targets, and trusted client devices.
This is both a security improvement and a way to avoid strange cross talk when I am testing tools or new services.

## Goals

- Management network for switches, APs, hypervisors, and core services  
- Lab network for vulnerable machines and experiments  
- Trusted network for family devices and normal traffic  

From there, I used UniFi to define VLAN only networks, tagged switch ports accordingly, and updated my core
router configuration to handle the additional subnets and firewall rules.

