# Palo Alto to FortiGate IKEv2 Site-to-Site IPsec VPN

This lab demonstrates a site-to-site IPsec VPN between a Palo Alto Networks firewall and a Fortinet FortiGate firewall.

The objective is to demonstrate multi-vendor VPN interoperability, routing, firewall policy, network segmentation, encryption, and systematic VPN troubleshooting in a private lab environment.

## Lab Objectives

This lab demonstrates:

- Palo Alto PAN-OS IPsec VPN configuration
- Fortinet FortiGate IPsec VPN configuration
- IKEv2
- IPsec Phase 1 and Phase 2
- Multi-vendor VPN interoperability
- Security policy
- Routing
- Tunnel interfaces
- Traffic selectors / Proxy IDs
- NAT considerations
- VPN verification
- Traffic logging
- Troubleshooting

## Lab Topology

```text
             SITE A                                  SITE B

         Linux Client                            Linux Server
         10.10.10.10                             10.20.20.10
          /24                                     /24
            |                                       |
            |                                       |
            v                                       v
       10.10.10.1                              10.20.20.1
    +---------------+                       +---------------+
    |  Palo Alto    |                       |   FortiGate   |
    |   PA-FW-01    |                       |   FGT-FW-01   |
    +---------------+                       +---------------+
            |                                       |
            |                                       |
      172.16.100.1/30                        172.16.100.2/30
            |                                       |
            +-------------------+-------------------+
                                |
                         IKEv2 / IPsec
```

## Network Addressing

| Device | Interface | Role | IP Address |
| --- | --- | --- | --- |
| Linux-01 | eth0 | Site A LAN | 10.10.10.10/24 |
| PA-FW-01 | ethernet1/2 | Site A LAN | 10.10.10.1/24 |
| PA-FW-01 | ethernet1/1 | WAN | 172.16.100.1/30 |
| FGT-FW-01 | port1 | WAN | 172.16.100.2/30 |
| FGT-FW-01 | port2 | Site B LAN | 10.20.20.1/24 |
| Linux-02 | eth0 | Site B LAN | 10.20.20.10/24 |

Protected networks:

```text
Site A: 10.10.10.0/24
Site B: 10.20.20.0/24
```

## VPN Parameters

The same cryptographic parameters must be configured on both firewalls.

### IKE Phase 1

```text
IKE Version:       IKEv2
Encryption:        AES-256
Authentication:    SHA-256
DH Group:          Group 14
Lifetime:          28800 seconds
Authentication:    Pre-Shared Key
```

The pre-shared key is intentionally not documented or stored in this repository.

### IPsec Phase 2

```text
Encryption:        AES-256
Authentication:    SHA-256
PFS:               Group 14
Lifetime:          3600 seconds
```

## Baseline Connectivity

Before configuring IPsec, basic connectivity should be verified.

### Palo Alto to FortiGate

The two WAN interfaces should communicate directly.

```text
PA-FW-01                              FGT-FW-01
172.16.100.1      <--- ICMP --->      172.16.100.2
```

Successful WAN connectivity confirms that the underlay network is operational before VPN troubleshooting begins.

## Site A Connectivity

Verify connectivity between the Site A client and Palo Alto LAN interface.

```text
Linux-01
10.10.10.10
     |
     v
10.10.10.1
PA-FW-01
```

The Linux client uses:

```text
Default Gateway: 10.10.10.1
```

## Site B Connectivity

Verify connectivity between the Site B host and FortiGate LAN interface.

```text
Linux-02
10.20.20.10
     |
     v
10.20.20.1
FGT-FW-01
```

The Linux host uses:

```text
Default Gateway: 10.20.20.1
```

At this stage, communication between the two protected networks should not yet depend on the VPN.

## Palo Alto Configuration

The Palo Alto firewall requires several components for a route-based IPsec VPN.

```text
IKE Crypto Profile
       |
       v
IKE Gateway
       |
       v
IPsec Crypto Profile
       |
       v
IPsec Tunnel
       |
       v
Tunnel Interface
       |
       v
Virtual Router
       |
       v
Security Policy
```

## Palo Alto Security Zones

Example zones:

```text
ethernet1/1  --->  UNTRUST
ethernet1/2  --->  TRUST
tunnel.1     --->  VPN
```

Conceptually:

```text
                UNTRUST
                   |
                   v
             ethernet1/1
                   |
             +-------------+
             |  Palo Alto  |
             +-------------+
               |         |
               |         |
          ethernet1/2  tunnel.1
               |         |
               v         v
             TRUST      VPN
```

## Palo Alto IKE Crypto Profile

Create an IKE crypto profile using the Phase 1 parameters.

Example:

```text
Name:            LAB-IKE-CRYPTO
Encryption:      AES-256
Authentication:  SHA-256
DH Group:        Group 14
Lifetime:        8 hours
```

## Palo Alto IKE Gateway

Create an IKE gateway.

Example:

```text
Name:              FGT-IKE-GW
IKE Version:       IKEv2
Local Interface:   ethernet1/1
Local IP:          172.16.100.1
Peer IP:           172.16.100.2
Authentication:    Pre-Shared Key
```

The actual pre-shared key must never be committed to Git.

## Palo Alto IPsec Crypto Profile

Create the Phase 2 crypto profile.

Example:

```text
Name:            LAB-IPSEC-CRYPTO
Encryption:      AES-256
Authentication:  SHA-256
DH Group / PFS:  Group 14
Lifetime:        1 hour
```

## Palo Alto Tunnel Interface

Create:

```text
tunnel.1
```

Assign the tunnel interface to:

```text
Security Zone: VPN
Virtual Router: Lab Virtual Router
```

For this lab, an IP address on the tunnel interface is not required for basic protected-network communication.

## Palo Alto IPsec Tunnel

Create an IPsec tunnel using:

```text
IKE Gateway:          FGT-IKE-GW
IPsec Crypto Profile: LAB-IPSEC-CRYPTO
Tunnel Interface:     tunnel.1
```

## Palo Alto Proxy IDs

For interoperability with the FortiGate, define the protected networks.

```text
Local Network:   10.10.10.0/24
Remote Network:  10.20.20.0/24
```

These networks must correspond with the traffic selectors configured on the FortiGate.

## Palo Alto Routing

The Palo Alto virtual router requires a route for the remote protected network.

Example:

```text
Destination:

10.20.20.0/24

Interface:

tunnel.1
```

Traffic destined for Site B will therefore be forwarded into the IPsec tunnel.

## Palo Alto Security Policy

Security policy must permit traffic between the TRUST and VPN zones.

Example:

```text
Rule Name: Site-A-to-Site-B

Source Zone:       TRUST
Destination Zone:  VPN

Source:
10.10.10.0/24

Destination:
10.20.20.0/24

Action:
ALLOW
```

A corresponding policy can be created for return traffic:

```text
Source Zone:       VPN
Destination Zone:  TRUST

Source:
10.20.20.0/24

Destination:
10.10.10.0/24

Action:
ALLOW
```

Logging should be enabled to make troubleshooting easier.

## FortiGate Configuration

The FortiGate requires corresponding VPN configuration.

The logical workflow is:

```text
Phase 1
   |
   v
Phase 2
   |
   v
IPsec Interface
   |
   v
Routing
   |
   v
Firewall Policy
```

## FortiGate Phase 1

Create an interface-based IPsec VPN.

Example parameters:

```text
IKE Version:       IKEv2
Interface:         port1
Remote Gateway:    172.16.100.1
Authentication:    Pre-Shared Key
Encryption:        AES-256
Authentication:    SHA-256
DH Group:          14
Lifetime:          28800
```

The pre-shared key must match the Palo Alto configuration but is intentionally omitted from this documentation.

## FortiGate Phase 2

Configure the Phase 2 selectors.

```text
Local Network:

10.20.20.0/24

Remote Network:

10.10.10.0/24
```

Crypto parameters:

```text
Encryption:       AES-256
Authentication:   SHA-256
PFS:              Group 14
Lifetime:         3600
```

Notice that the local and remote networks are reversed relative to the Palo Alto configuration.

From the Palo Alto perspective:

```text
Local:   10.10.10.0/24
Remote:  10.20.20.0/24
```

From the FortiGate perspective:

```text
Local:   10.20.20.0/24
Remote:  10.10.10.0/24
```

## FortiGate Routing

Create a route to the Site A protected network.

```text
Destination:

10.10.10.0/24

Interface:

IPsec VPN Interface
```

No Internet next-hop gateway is required for a route-based VPN interface.

## FortiGate Firewall Policy

Create a policy permitting Site B traffic into the VPN.

Example:

```text
Source Interface:       Site-B-LAN
Destination Interface:  IPsec Tunnel

Source:
10.20.20.0/24

Destination:
10.10.10.0/24

Action:
ACCEPT

NAT:
DISABLED
```

Create the reverse policy:

```text
Source Interface:       IPsec Tunnel
Destination Interface:  Site-B-LAN

Source:
10.10.10.0/24

Destination:
10.20.20.0/24

Action:
ACCEPT

NAT:
DISABLED
```

NAT should normally not be applied to traffic between the protected networks in this lab.

## Expected VPN Traffic Flow

Once the VPN is established:

```text
Linux-01
10.10.10.10
     |
     v
Palo Alto TRUST
     |
     v
Security Policy
     |
     v
Route to tunnel.1
     |
     v
IPsec Encryption
     |
     |
=====+=====================================
          IKEv2 / IPsec Tunnel
=========================================== 
                              |
                              v
                       IPsec Decryption
                              |
                              v
                       FortiGate Policy
                              |
                              v
                         Site B LAN
                              |
                              v
                         Linux-02
                       10.20.20.10
```

## Validation

After configuration, test from Site A:

```bash
ping 10.20.20.10
```

Then test from Site B:

```bash
ping 10.10.10.10
```

Successful bidirectional communication verifies:

- LAN connectivity
- Routing
- Firewall policy
- IKE negotiation
- IPsec negotiation
- Traffic selectors
- Encryption
- Decryption
- Return routing

## Palo Alto Verification

Verify that the IKE security association is established.

Useful PAN-OS operational commands include:

```text
show vpn ike-sa
```

Verify IPsec security associations:

```text
show vpn ipsec-sa
```

The Palo Alto traffic logs can also be used to confirm that traffic matches the expected security policy.

## FortiGate Verification

Useful FortiGate VPN commands include:

```text
get vpn ipsec tunnel summary
```

Additional tunnel information:

```text
diagnose vpn tunnel list
```

These commands help determine whether the tunnel is established and whether encrypted/decrypted packet counters are increasing.

## Troubleshooting Methodology

VPN troubleshooting should be performed in layers.

```text
1. Underlay Connectivity
          |
          v
2. IKE Phase 1
          |
          v
3. IPsec Phase 2
          |
          v
4. Routing
          |
          v
5. Security Policy
          |
          v
6. NAT
          |
          v
7. End-to-End Traffic
```

## 1. Verify Underlay Connectivity

Before troubleshooting IPsec, verify:

```text
172.16.100.1 <----> 172.16.100.2
```

If the peers cannot reach each other, IKE cannot establish.

## 2. Verify IKE Phase 1

Check:

- Peer IP addresses
- IKE version
- Pre-shared key
- Encryption algorithm
- Authentication algorithm
- DH group
- Lifetime

A Phase 1 failure means the peers have not established an IKE security association.

## 3. Verify IPsec Phase 2

If Phase 1 succeeds but Phase 2 fails, check:

- Encryption
- Authentication
- PFS
- Lifetime
- Local subnet
- Remote subnet
- Proxy IDs
- Traffic selectors

Multi-vendor VPNs frequently expose traffic-selector mismatches.

## 4. Verify Routing

Both firewalls require routes to the remote protected network.

```text
Palo Alto:

10.20.20.0/24
     |
     v
tunnel.1


FortiGate:

10.10.10.0/24
     |
     v
IPsec Interface
```

Without the correct routes, traffic may never enter the VPN.

## 5. Verify Security Policy

The tunnel can be fully established while user traffic is still blocked by firewall policy.

Verify policy in both directions.

## 6. Verify NAT

Unexpected NAT can prevent VPN traffic from matching the configured traffic selectors.

For this lab:

```text
10.10.10.0/24 <----> 10.20.20.0/24
```

should communicate without address translation.

## 7. Verify End-to-End Traffic

Finally, validate:

```text
10.10.10.10

        |
        | encrypted
        v

      VPN

        |
        | decrypted
        v

10.20.20.10
```

Check packet counters and traffic logs while generating test traffic.

## Deliberate Failure Testing

Once the VPN works correctly, the lab can be made more useful by intentionally introducing configuration errors.

Examples include:

### Incorrect Phase 1 Encryption

Change one peer from:

```text
AES-256
```

to an incompatible proposal and observe the resulting IKE failure.

### Incorrect Pre-Shared Key

Temporarily configure different authentication values and examine the resulting logs.

Never document the actual keys.

### Incorrect Phase 2 Selector

For example, configure:

```text
10.20.30.0/24
```

instead of:

```text
10.20.20.0/24
```

and observe the resulting Phase 2 behavior.

### Missing Route

Remove the remote-network route while leaving the VPN established.

This demonstrates an important troubleshooting principle:

> A VPN being UP does not prove that application traffic can traverse it.

### Missing Security Policy

Disable a firewall policy and compare the VPN status with the resulting traffic logs.

## Key Lessons

### Multi-Vendor Standards Matter

Palo Alto and Fortinet use different interfaces and terminology, but both implement standard IKE and IPsec technologies.

Understanding the protocol is more important than memorizing one vendor's GUI.

### Tunnel Status Is Only One Layer

An established tunnel does not guarantee end-to-end connectivity.

Routing, firewall policy, NAT, and endpoint configuration must also be correct.

### Local and Remote Are Relative

Traffic selectors are described from each firewall's perspective.

```text
Palo Alto

Local  = Site A
Remote = Site B


FortiGate

Local  = Site B
Remote = Site A
```

This is a common source of configuration errors.

### Troubleshoot the Packet Path

The most reliable approach is to follow the traffic:

```text
Host
 |
Firewall
 |
Policy
 |
Route
 |
Encryption
 |
VPN
 |
Decryption
 |
Route
 |
Policy
 |
Host
```

## Future Enhancements

This lab can later be expanded to include:

- Multiple protected networks
- Multiple Phase 2 selectors
- Dynamic routing over IPsec
- Tunnel monitoring
- Redundant VPN tunnels
- Dual WAN connectivity
- FortiGate SD-WAN integration
- Palo Alto path monitoring
- NAT across VPN
- Centralized logging
- VPN monitoring
- Configuration backup
- API-based VPN auditing
- Python automation
- Automated tunnel-health reporting

## Security and Lab Environment

This lab is performed in a private, non-production environment.

Screenshots and documentation may include internal lab hostnames, RFC1918 private IP addresses, firewall names, interface names, VLAN IDs, and other lab-specific information where those details help demonstrate the configuration or troubleshooting process.

Passwords, VPN pre-shared keys, API keys, authentication tokens, private keys, certificates, public IP addresses, and other sensitive authentication material are not published.

Any configuration output should be reviewed before being committed to this repository.

No employer, customer, or production configuration data is included.
