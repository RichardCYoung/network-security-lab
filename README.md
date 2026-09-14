# Network Security Lab

Hands-on network security lab covering Palo Alto Networks, Fortinet FortiGate, Cisco ASA, firewall policy, NAT, network segmentation, VPN technologies, logging, security auditing, and troubleshooting.

This repository documents practical network security work performed in a private, non-production lab environment.

The primary focus is Network Security Engineering: designing, implementing, validating, and troubleshooting security controls across multi-vendor network infrastructure.

## Lab Overview

The security lab includes multiple firewall platforms:

- Palo Alto Networks firewalls
- Fortinet FortiGate firewalls
- Cisco ASA
- Managed Ethernet switching
- VMware / virtual infrastructure
- Linux and Windows systems
- Virtual network appliances
- EVE-NG network emulation

The environment is used to build and test security scenarios without affecting production infrastructure.

## Areas of Focus

Lab work includes:

- Firewall security policy
- Security zones
- Network segmentation
- Inter-VLAN security
- Network Address Translation (NAT)
- Site-to-site IPsec VPN
- IKEv2
- Remote-access concepts
- Routing through firewalls
- High availability concepts
- Logging and traffic analysis
- Firewall policy auditing
- Multi-vendor interoperability
- Security troubleshooting
- Network automation

## Lab Architecture

A typical lab security topology consists of multiple security zones separated by firewall policy.

```text
                         INTERNET / WAN
                              |
                              |
                       +-------------+
                       |  Firewall   |
                       +-------------+
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
            TRUST           SERVER           DMZ
              |               |               |
              v               v               v
          User / Admin     Application      Public /
            Systems          Servers       Test Server


                        MANAGEMENT
                            |
                            v
                    Network Management
```

The goal is to control traffic between security zones rather than relying only on VLAN separation.

## Multi-Vendor Firewall Lab

The lab provides hands-on experience with multiple firewall operating systems.

```text
                  Network Security Lab
                         |
        +----------------+----------------+
        |                |                |
        v                v                v

   Palo Alto         FortiGate        Cisco ASA
      PAN-OS           FortiOS           ASA
        |                |                |
        +----------------+----------------+
                         |
                         v
                Multi-Vendor Testing
```

This allows equivalent security concepts to be implemented and compared across different vendors.

## Palo Alto Networks

The Palo Alto portion of the lab focuses on application-aware security policy and zone-based firewall design.

Lab topics include:

- PAN-OS administration
- Security zones
- Layer-3 interfaces
- Virtual routers
- Security policies
- NAT policies
- Application-based policy
- Service objects
- Address objects
- Logging
- Traffic monitoring
- IPsec VPN
- IKE gateways
- Tunnel interfaces
- Routing
- Policy troubleshooting
- Packet-flow troubleshooting

Example zone design:

```text
                    UNTRUST
                       |
                       v
                +-------------+
                | Palo Alto   |
                |  Firewall   |
                +-------------+
                  |    |    |
          +-------+    |    +-------+
          |            |            |
          v            v            v
        TRUST        SERVER        DMZ
```

Security policy determines which applications and services are permitted between these zones.

## Fortinet FortiGate

The Fortinet portion of the lab focuses on FortiGate firewall policy, routing, NAT, VPN, and network security services.

Lab topics include:

- FortiOS administration
- Firewall policies
- Interfaces and zones
- Address objects
- Service objects
- NAT
- Static routing
- Policy routing
- IPsec VPN
- IKEv2
- SD-WAN concepts
- Logging
- Traffic troubleshooting
- Security profiles
- High availability concepts

Example:

```text
                  WAN
                   |
                   v
             +-----------+
             | FortiGate |
             +-----------+
               |       |
               |       |
               v       v
             USERS   SERVERS
```

## Cisco ASA

Cisco ASA provides an additional platform for testing traditional stateful firewall concepts and comparing them with newer next-generation firewall platforms.

Lab topics include:

- ASA interfaces
- Security levels
- Access control lists
- Network objects
- Object groups
- NAT
- Static routing
- VPN concepts
- Logging
- Packet-tracer
- Connectivity troubleshooting

The ASA is particularly useful for understanding traditional firewall processing and legacy environments that continue to operate alongside newer platforms.

## Security Zones and Segmentation

Network segmentation is a major focus of the lab.

Example:

```text
                     FIREWALL
                        |
       +----------------+----------------+
       |                |                |
       v                v                v

     USERS            SERVERS           DMZ
    VLAN 10           VLAN 20         VLAN 30
       |                |                |
       |                |                |
       +-------- Firewall Policy --------+
```

VLANs provide Layer-2 segmentation.

The firewall provides security enforcement between those network segments.

Example policy:

```text
Source      Destination    Application / Service    Action
----------------------------------------------------------
USERS       INTERNET       Web                     ALLOW
USERS       SERVERS        HTTPS                   ALLOW
USERS       SERVERS        SSH                     DENY
ADMIN       SERVERS        SSH                     ALLOW
DMZ         INTERNAL       Any                     DENY
SERVERS     DNS            DNS                     ALLOW
```

This demonstrates least-privilege access rather than unrestricted inter-VLAN routing.

## Network Address Translation

The lab is also used to test different NAT scenarios.

Examples include:

- Source NAT
- Destination NAT
- Static NAT
- Port translation
- Internet access
- Published services
- NAT exemption for VPN traffic

Typical source NAT traffic flow:

```text
Internal Host
10.x.x.x
    |
    v
Firewall
    |
    | Source NAT
    |
    v
WAN Interface
    |
    v
Internet
```

NAT troubleshooting is performed together with routing and security-policy validation.

## Site-to-Site VPN

One of the key multi-vendor lab scenarios is site-to-site IPsec VPN connectivity.

Example:

```text
        Site A                              Site B

    10.10.10.0/24                       10.20.20.0/24
          |                                   |
          v                                   v
   +---------------+                   +---------------+
   |  Palo Alto    |                   |   FortiGate   |
   +---------------+                   +---------------+
          |                                   |
          |                                   |
          +========= IKEv2 / IPsec ===========+
```

This provides hands-on experience with multi-vendor VPN interoperability.

VPN configuration and troubleshooting includes:

- IKE Phase 1
- IPsec Phase 2
- Encryption algorithms
- Authentication
- Diffie-Hellman groups
- Security associations
- Proxy IDs / traffic selectors
- Tunnel interfaces
- Routing
- Security policy
- NAT exemption
- Tunnel monitoring
- Logging

## VPN Troubleshooting

A VPN tunnel being established does not necessarily mean application traffic will work.

The complete path must be validated:

```text
Source Host
     |
     v
Source Firewall
     |
     v
Security Policy
     |
     v
Routing
     |
     v
IPsec Encryption
     |
     v
====================
    VPN Tunnel
====================
     |
     v
IPsec Decryption
     |
     v
Destination Firewall
     |
     v
Security Policy
     |
     v
Destination Host
```

Common problems include:

- Phase 1 mismatch
- Phase 2 mismatch
- Incorrect traffic selectors
- Missing routes
- Incorrect security policy
- NAT applied unexpectedly
- Asymmetric routing
- MTU issues
- Incorrect encryption parameters

## Firewall Troubleshooting Methodology

Firewall troubleshooting should follow the packet through the entire system.

```text
Source
   |
   v
Ingress Interface
   |
   v
Security Zone
   |
   v
Routing Decision
   |
   v
NAT
   |
   v
Security Policy
   |
   v
VPN / Inspection
   |
   v
Egress Interface
   |
   v
Destination
```

The exact packet-processing order varies between firewall platforms, so vendor-specific behavior must also be considered.

## Logging and Traffic Analysis

Logs are used extensively when troubleshooting firewall behavior.

Examples include:

- Allowed traffic
- Denied traffic
- Policy matches
- NAT translations
- VPN negotiation
- Authentication
- System events
- Interface events
- Routing changes

A firewall policy that appears correct should still be validated against actual traffic logs.

## Multi-Vendor Comparison

The lab makes it possible to compare similar concepts across platforms.

| Security Concept | Palo Alto | Fortinet | Cisco ASA |
| --- | --- | --- | --- |
| Firewall Rules | Security Policy | Firewall Policy | ACL |
| Interfaces | Interfaces / Zones | Interfaces / Zones | Named Interfaces |
| Network Objects | Address Objects | Address Objects | Network Objects |
| NAT | NAT Policy | Firewall/NAT Configuration | NAT Rules |
| VPN | IKE/IPsec | IKE/IPsec | IKE/IPsec |
| Routing | Virtual Router | Routing Table | Routing Table |
| Traffic Testing | Logs / Packet Tools | Logs / Debug Flow | Packet-Tracer |
| Management | PAN-OS | FortiOS | ASA CLI/ASDM |

The terminology differs, but the fundamental network-security principles remain consistent.

## Network Security Automation

The lab will also be integrated with the Network Automation project.

Potential automated checks include:

```text
Firewall Security Audit
        |
        +---- Disabled Rules
        |
        +---- Any / Any Rules
        |
        +---- Rules Without Logging
        |
        +---- Unused Objects
        |
        +---- NAT Rules
        |
        +---- Interface Status
        |
        +---- VPN Status
        |
        +---- Security Policy Review
```

Example future output:

```text
NETWORK SECURITY AUDIT
========================================

Device: LAB-FW-01
Platform: Palo Alto Networks

Security Policy
----------------------------------------
Rules:                    32
Disabled Rules:            2
Broad Access Rules:        1
Rules Without Logging:     3

VPN
----------------------------------------
Tunnels:                   3
Up:                        2
Down:                      1

Audit
----------------------------------------
PASS:                     18
WARNING:                   4
FAIL:                      1
```

This combines network security knowledge with Python, APIs, and infrastructure automation.

## Integration With the Lab

The security environment complements other projects in this GitHub portfolio.

```text
                    Physical Network
                           |
                           v
                  Network Security
                 Palo Alto / Fortinet
                           |
              +------------+------------+
              |                         |
              v                         v
        Virtualization              EVE-NG
       VMware / Proxmox          Network Lab
              |                         |
              +------------+------------+
                           |
                           v
                       Kubernetes
                           |
                           v
                         NetBox
                           |
                           v
                  Network Automation
```

This provides an environment for testing infrastructure across networking, security, virtualization, Linux, Kubernetes, and automation.

## Repository Structure

```text
network-security-lab/
│
├── README.md
│
├── palo-alto/
│   └── troubleshooting.md
│
├── fortinet/
│   └── troubleshooting.md
│
├── cisco-asa/
│   └── troubleshooting.md
│
├── vpn/
│   └── site-to-site-vpn.md
│
├── segmentation/
│   └── security-zones.md
│
├── automation/
│   └── firewall-audit.py
│
└── screenshots/
```

The repository will expand as additional lab scenarios are built and documented.

## Future Lab Work

Planned additions include:

- Palo Alto security-policy audit
- FortiGate firewall-policy audit
- Palo Alto to FortiGate IPsec VPN
- VPN failure and troubleshooting scenarios
- Network segmentation lab
- DMZ implementation
- NAT troubleshooting
- Firewall logging examples
- High availability testing
- Firewall configuration backup
- REST API automation
- Python security-policy auditing
- NetBox integration
- Centralized logging
- Security monitoring
- Configuration compliance reporting

## Security and Lab Environment

This repository contains examples, documentation, configurations, diagrams, and screenshots from a private, non-production lab environment.

Screenshots and documentation may include internal lab hostnames, private IP addressing, device names, VLAN IDs, virtual machine names, firewall objects, and other lab-specific details where they help demonstrate the environment or troubleshooting scenario.

Credentials, passwords, VPN pre-shared keys, API keys, authentication tokens, private keys, certificates, public IP addresses, and other sensitive information are not intentionally published.

Firewall configurations and screenshots are reviewed before publication to avoid exposing authentication information or other security-sensitive material.

All systems and configurations shown are part of a personal lab environment unless explicitly identified as generalized examples.

No employer, customer, or production configuration data is included.
