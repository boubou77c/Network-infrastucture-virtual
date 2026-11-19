## 1. Project Overview 

For an academic project, with 4 persons, we need to build a virtual network infrastructure for a fictive enterprise.

The objective was to build a segmented,secured and redundant network environment by integrating virtual network devices, firewalls, switches, servers and client Workstations.

The infrastructure was deployed using GNS3 for network simulation and VMware Workstation Pro for virtual machine hosting.

The project covers several key areas of network and system administration, including VLAN Segmentation, inter-VLAN routing, firewall configuration, network redundancy, server integration and traffic analysis.

## 2. Network Architecture

### Infrastructure Components

| Hostname | Technology | Role |
|---|---|---|
| FW01 | pfSense | Firewall, NAT, inter-VLAN routing and DHCP relay |
| SW1-CORE | Cisco IOSvL2 | Core switching, VLANs and network management |
| SW2-ACCESS | Cisco IOSvL2 | Access switching and Layer 2 redundancy |
| DC01 | Windows Server 2025 | Active Directory, DNS and DHCP |
| PC-ADMIN01 | Windows 11 Enterprise | Administrative workstation |
| PC-USER01 | Windows 11 Enterprise | Domain user workstation |
| WEB01 | Ubuntu Server | Web server hosted in the DMZ |
| MONLOG01 | Ubuntu Server | Centralized logging and Zabbix monitoring |

## 3. Network Segmentation & IP Addressing

The infrastructure is divided into five functional VLANs and one dedicated parking VLAN.

| VLAN | Name | Network | Gateway |
|---|---|---|---|
| 10 | ADMIN | 10.30.10.0/24 | 10.30.10.1 |
| 20 | USERS | 10.30.20.0/24 | 10.30.20.1 |
| 30 | SERVERS | 10.30.30.0/24 | 10.30.30.1 |
| 40 | DMZ | 10.30.40.0/24 | 10.30.40.1 |
| 50 | MGMT | 10.30.50.0/24 | 10.30.50.1 |
| 999 | PARKING / NATIVE | Non-routed | None |

### VLAN Purposes

- **VLAN 10 – ADMIN:** Dedicated administrative workstation network.
- **VLAN 20 – USERS:** Standard user workstation network.
- **VLAN 30 – SERVERS:** Internal infrastructure servers.
- **VLAN 40 – DMZ:** Isolated network hosting the Web server.
- **VLAN 50 – MGMT:** Dedicated network for Cisco switch management.
- **VLAN 999 – PARKING:** Non-routed VLAN used for unused switch ports and native VLAN configuration.

### Switching Configuration

- VLAN creation and assignment.
- IEEE 802.1Q trunk configuration.
- Access port configuration.
- Spanning Tree Protocol (STP).
- EtherChannel / LACP for switch interconnection.
- Restricted VLANs on trunk links.
- Dedicated management VLAN.
- Unused port isolation using VLAN 999.
- PortFast and BPDU Guard where supported.

## 4. Firewall & Network Security – pfSense

pfSense acts as the central security gateway of the infrastructure.

All inter-VLAN communications pass through pfSense to enforce access control and network isolation.

### Implemented Features

- Inter-VLAN routing.
- Network Address Translation (NAT).
- Stateful firewall rules.
- Default-deny security policy.
- DHCP relay towards DC01.
- Traffic filtering between VLANs.
- DMZ isolation.
- WAN port forwarding for Web services.
- Firewall event logging.

### Network Security Policy

The infrastructure follows the principle of least privilege.

Only explicitly authorized communications are permitted between network segments.

| Source | Destination | Policy |
|---|---|---|
| ADMIN | pfSense | HTTPS / ICMP allowed |
| ADMIN | Cisco switches | SSH / ICMP allowed |
| ADMIN | Servers / DMZ | Administrative access according to requirements |
| USERS | DC01 | Required Active Directory services allowed |
| USERS | WEB01 | HTTP / HTTPS allowed |
| USERS | ADMIN / MGMT | Blocked and logged |
| DMZ | Internal LAN | Blocked and logged |
| Cisco / pfSense | MONLOG01 | Syslog allowed |
| MONLOG01 | Monitored devices | SNMPv3 / monitoring traffic allowed |

A final default-deny rule blocks and logs unauthorized traffic.

## 5. Cisco Switching & Layer 2 Security

The infrastructure uses two Cisco IOSvL2 switches:

- **SW1-CORE:** Core Layer 2 switching.
- **SW2-ACCESS:** Access switching and endpoint connectivity.

Both switches are managed through Switched Virtual Interfaces (SVIs) in VLAN 50 (MGMT).

### VLAN & Trunk Configuration

- Creation and assignment of VLANs.
- IEEE 802.1Q trunk configuration.
- Explicit VLAN pruning on trunk interfaces.
- Native VLAN reassignment to VLAN 999.
- Access port configuration.
- EtherChannel using LACP (IEEE 802.3ad).
- Spanning Tree Protocol (STP) for loop prevention.

### Layer 2 Security Hardening

Several security mechanisms were implemented to reduce Layer 2 attack surfaces:

- Unused switch ports assigned to VLAN 999 and administratively disabled.
- PortFast enabled on host-facing access ports.
- BPDU Guard enabled on supported endpoint interfaces.
- Trunk interfaces restricted to operational VLANs (10, 20, 30, 40 and 50).
- VLAN 999 configured as the native VLAN on trunk links.

### Secure Remote Administration

Cisco management access is restricted to the administrative network.

- SSH version 2 exclusively.
- RSA key generation.
- Local user authentication.
- VTY access restricted through an ACL to the ADMIN subnet (10.30.10.0/24).
- Telnet disabled.
- EXEC session timeouts.
- Configuration persistence using `write memory`.

These measures protect network administration against unauthorized access and plaintext credential interception.

## 6. Network Redundancy – LAGG Failover

A Link Aggregation Group (LAGG) configured in Failover mode provides link redundancy between pfSense and SW1-CORE.

### Implementation

- Multiple physical interfaces grouped into one logical interface.
- One active link and one standby link.
- Automatic failover if the active interface becomes unavailable.
- VLAN interfaces configured over the LAGG.

The failover mechanism was tested by disconnecting the active link and observing the recovery of network connectivity.

### Switch Interconnection

STP and LACP / EtherChannel are used for the interconnection between the Cisco switches.

These mechanisms address Layer 2 redundancy and loop prevention, while the pfSense LAGG provides firewall-to-switch link redundancy.

## 7. Windows Server Infrastructure – Active Directory

DC01 runs Windows Server 2025 and provides centralized identity and network services.

### Implemented Services

- Active Directory Domain Services (AD DS).
- DNS.
- DHCP.
- Domain user and computer management.
- Organizational Units (OU).
- Group Policy Objects (GPO).

### Active Directory Domain

`gcs1.guardia.test`

### Organizational Structure

- GCS1-Admins
- GCS1-Users
- GCS1-Computers

### Group Policy – Security Hardening

At least three security policies were implemented and validated, including:

- Automatic session locking.
- Windows Defender Firewall configuration for the Domain profile.
- Password and account lockout policies or additional approved security hardening.

Policy application was verified using:

- `gpupdate /force`
- `gpresult`

### DHCP Configuration

The DHCP scope for VLAN 20 distributes:

- IP addresses: 10.30.20.100 – 10.30.20.199
- Subnet mask: 255.255.255.0
- Default gateway: 10.30.20.1
- DNS server: 10.30.30.10

DHCP requests are relayed by pfSense to DC01.

## 8. DMZ – Web Server

WEB01 runs Ubuntu Server and hosts a Web service using **Nginx**.

| Parameter | Value |
|---|---|
| Hostname | WEB01 |
| Operating System | Ubuntu Server |
| Web Server | Nginx |
| VLAN | 40 – DMZ |
| IP Address | 10.30.40.10/24 |
| Gateway | 10.30.40.1 |

### Implementation

- Nginx Web server deployment.
- HTTP service (TCP 80).
- HTTPS service (TCP 443).
- Static IP configuration.
- External access through pfSense port forwarding.
- Network isolation through DMZ firewall policies.

## 9. Centralized Logging – Rsyslog

MONLOG01 runs Ubuntu Server and provides centralized log collection through Rsyslog.

### Server Configuration

| Parameter | Value |
|---|---|
| Hostname | MONLOG01 |
| Operating System | Ubuntu Server |
| IP Address | 10.30.30.20/24 |
| VLAN | 30 – SERVERS |
| Protocol | Syslog |
| Listening Port | UDP 514 |

### Log Collection

Rsyslog receives timestamped system and security events from:

- SW1-CORE – Cisco IOSvL2.
- SW2-ACCESS – Cisco IOSvL2.
- FW01 – pfSense.

Incoming logs are automatically organized by source device IP under:

```text
/var/log/remote/
```

Example:

```text
/var/log/remote/
├── 10.30.50.11.log
└── 10.30.50.12.log
```

Centralized logging provides a unified audit trail for:

- Firewall rule verification.
- Security event investigation.
- Network troubleshooting.
- Monitoring infrastructure activity.

---

## 10. Infrastructure Monitoring – Zabbix & SNMPv3

Zabbix is deployed on MONLOG01 to provide centralized monitoring of network devices, servers and Web services.

The monitoring infrastructure provides real-time visibility into device availability, network interface status and service health.

### Cisco Switch Monitoring – SNMPv3

Both Cisco switches are monitored using authenticated and encrypted SNMPv3.

Monitoring includes:

- Device availability.
- Interface operational status.
- Network interface utilization.
- Device health indicators.

SNMPv3 provides secure monitoring communication instead of using unencrypted SNMPv1/v2c community strings.

### Server & Service Monitoring

Zabbix monitors several infrastructure components:

| Monitored Component | Monitoring Method |
|---|---|
| SW1-CORE | SNMPv3 |
| SW2-ACCESS | SNMPv3 |
| DC01 | Zabbix Agent (TCP 10050) |
| WEB01 | HTTP service check |
| Network nodes | ICMP availability checks |

### Dashboard & Alerting

A custom Zabbix dashboard was created with four operational indicators.

Automated problem alerting was also validated by simulating a service outage.

This provides centralized visibility into infrastructure availability and helps identify service interruptions.

## 10. Infrastructure Monitoring – Zabbix

Zabbix is deployed on MONLOG01 to provide centralized monitoring of network devices and infrastructure services.

### Monitoring Scope

The monitoring implementation includes:

- ICMP availability monitoring.
- Cisco switch interface monitoring through SNMPv3.
- WEB01 HTTP service monitoring.
- DC01 availability monitoring.

### Monitoring Features

- Infrastructure availability monitoring.
- Network interface monitoring.
- Monitoring dashboards.
- Performance and status indicators.
- Alert configuration and problem detection.

A monitoring dashboard with at least four indicators was implemented, together with a demonstrated failure alert.

SNMPv3 is used for secure communication with the monitored Cisco equipment.

## 11. Network Testing & Validation

The infrastructure was validated through functional, connectivity and security tests.

### Network Tests

- VLAN connectivity.
- Gateway reachability.
- DHCP address allocation.
- DNS resolution.
- Domain authentication.
- GPO application.
- Internet connectivity.
- Web server accessibility.
- LAGG failover.

### Security Tests

| Test | Expected Result |
|---|---|
| USERS → ADMIN | Blocked and logged |
| USERS → WEB01 (80/443) | Allowed |
| DMZ → DC01 | Blocked and logged |
| ADMIN → SW1/SW2 via SSH | Allowed |
| USERS → SW1/SW2 via SSH | Blocked |
| Telnet administration | Disabled |
| Cisco / pfSense → MONLOG01 | Syslog received |
| Zabbix → Cisco via SNMPv3 | Monitoring operational |
| WAN → WEB01 (80/443) | Allowed through configured port forwarding |

### Traffic Analysis – Wireshark

Wireshark was used to inspect and validate network traffic, including:

- ARP and DHCP exchanges.
- DHCP Discover, Offer, Request and ACK.
- IEEE 802.1Q VLAN tags.
- DNS traffic.
- TCP HTTP/HTTPS sessions.
- Authorized and blocked network communications.

## 12. Backup & Documentation

Configuration backups and recovery points were maintained throughout the project.

- Cisco configurations saved using `write memory`.
- pfSense configuration exported using `config.xml`.
- GNS3 snapshots at key project milestones.
- VMware virtual machine snapshots.
- Network topology documentation using draw.io.
- Technical report and validation evidence.

## 13. Technologies & Tools

| Category | Technologies |
|---|---|
| Network Simulation | GNS3 |
| Virtualization | VMware Workstation Pro |
| Firewall | pfSense |
| Switching | Cisco IOSvL2 |
| Network Protocols | VLAN, 802.1Q, STP, LACP, EtherChannel |
| Network Security | NAT, Firewall Rules, SSH v2 |
| Directory Services | Active Directory, DNS, DHCP, GPO |
| Operating Systems | Windows Server 2025, Windows 11, Ubuntu Server |
| Web Services | HTTP / HTTPS |
| Logging | Rsyslog, Syslog |
| Monitoring | Zabbix, SNMPv3, ICMP |
| Network Analysis | Wireshark |



## 14. Skills Developed

- Enterprise network architecture design.
- Network segmentation and VLAN management.
- Cisco Layer 2 switching and administration.
- Firewall configuration and traffic filtering.
- Inter-VLAN routing and NAT.
- Network redundancy and failover.
- Active Directory and Windows Server administration.
- DNS, DHCP and Group Policy management.
- Linux server administration.
- Centralized logging with Rsyslog.
- Infrastructure monitoring with Zabbix.
- SNMPv3 configuration and network monitoring.
- Network troubleshooting and packet analysis.
- Technical documentation and validation.

## 15. Project Context

**Academic Group Project – GCS1-UE2: Security ISR**

**Academic year:** 2025–2026

This project was developed collaboratively as part of an academic networking and infrastructure security module.

The repository documents the architecture, implementation, configurations and validation of the GUARDIA-NET virtual enterprise infrastructure.

