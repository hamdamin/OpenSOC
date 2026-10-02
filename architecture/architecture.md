# OpenSOC — Lab Architecture

This document describes the architecture, network topology and data flow of the OpenSOC laboratory environment.

The lab is designed to simulate a small enterprise network containing an Active Directory environment, Windows endpoints, centralized security monitoring and a network security gateway.

---

## 1. Architecture Overview

The OpenSOC laboratory is built as an isolated virtualized enterprise environment.

The main components are:

* pfSense — network gateway and firewall
* Windows Server 2022 — Active Directory Domain Controller
* Windows 11 — domain-joined endpoint
* Ubuntu Server — Wazuh SIEM
* Sysmon — Windows endpoint telemetry
* Wazuh Agent — endpoint log collection
* Wazuh Manager — centralized event processing and detection
* Wazuh Dashboard — security monitoring and investigation interface

The laboratory network uses the private address space:

```text
10.10.10.0/24
```

---

## 2. Network Topology

```text
                              Internet
                                  │
                                  │ WAN
                           ┌──────▼──────┐
                           │   pfSense   │
                           │   Gateway   │
                           │ 10.10.10.1  │
                           └──────┬──────┘
                                  │
                         LAN: 10.10.10.0/24
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
              │                   │                   │
       ┌──────▼──────┐     ┌──────▼──────┐     ┌──────▼──────┐
       │ Windows DC  │     │ Windows 11  │     │   Ubuntu     │
       │ Server 2022 │     │  Endpoint   │     │    Server    │
       │10.10.10.10  │     │    DHCP     │     │10.10.10.20  │
       └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
              │                   │                   │
           Sysmon              Sysmon          Wazuh Manager
              │                   │                   │
        Wazuh Agent         Wazuh Agent       Wazuh Dashboard
              │                   │                   │
              └───────────────────┴───────────────────┘
```

---

## 3. Network Configuration

| Component           |      IP Address | Role                       |
| ------------------- | --------------: | -------------------------- |
| pfSense             |    `10.10.10.1` | Default gateway / firewall |
| Windows Server 2022 |   `10.10.10.10` | Domain Controller          |
| Windows 11          |            DHCP | Domain client              |
| Ubuntu / Wazuh      |   `10.10.10.40` | SIEM                       |
| Network             | `10.10.10.0/24` | Isolated SOC LAN           |

The pfSense DHCP server is configured to provide addresses from:

```text
10.10.10.100 - 10.10.10.200
```

Static infrastructure addresses are reserved outside the DHCP range.

---

## 4. Component Roles

### 4.1 pfSense

pfSense acts as the network gateway and firewall for the laboratory.

Responsibilities include:

* Routing between the laboratory and WAN
* Network isolation
* Firewall policy enforcement
* DHCP
* Providing visibility into network traffic
* Controlling which services can communicate with the outside network

The LAN interface is:

```text
10.10.10.1/24
```

---

### 4.2 Windows Server 2022

The Windows Server acts as the laboratory's Active Directory Domain Controller.

Responsibilities include:

* Active Directory Domain Services
* User and computer accounts
* Domain authentication
* Kerberos authentication
* LDAP services
* DNS for the domain
* Centralized Windows security events

The server is configured without a graphical desktop environment and is administered through PowerShell.

IP address:

```text
10.10.10.10
```

Security telemetry is collected using:

* Windows Security Event Log
* Sysmon
* Wazuh Agent
* Windows OpenSSH telemetry where applicable

---

### 4.3 Windows 11 Endpoint

The Windows 11 machine represents an employee workstation in the simulated enterprise.

It is joined to the Active Directory domain and generates endpoint telemetry that can be analyzed by the SOC.

Telemetry sources include:

* Windows Security logs
* Sysmon
* Wazuh Agent

The endpoint receives its address through DHCP.

---

### 4.4 Ubuntu Server / Wazuh

The Ubuntu Server hosts the centralized Wazuh deployment.

Wazuh is responsible for:

* Receiving endpoint telemetry
* Processing security events
* Applying detection rules
* Generating alerts
* Mapping detections to MITRE ATT&CK
* Providing security monitoring through the dashboard

IP address:

```text
10.10.10.20
```

The Wazuh deployment currently operates as the central SIEM/security monitoring platform for the laboratory.

---

### 4.5 Sysmon

Sysmon provides detailed Windows endpoint telemetry that is not available from basic Windows authentication logs alone.

Important telemetry includes:

* Process creation
* Process termination
* Network connections
* File creation
* Registry activity
* Driver activity
* DNS activity


---

### 4.6 Wazuh Agent

The Wazuh Agent runs on monitored Windows systems.

Its responsibilities include:

* Collecting Windows event logs
* Collecting Sysmon telemetry
* Monitoring endpoint activity
* Forwarding telemetry to the Wazuh Manager

Current monitored systems:

```text
Windows Server 2022
Windows 11
```

---

## 5. Telemetry Flow

The general telemetry flow is:

```text
Windows Endpoint
       │
       ├── Windows Security Events
       │
       ├── Sysmon Events
       │
       │
       ▼
 Wazuh Agent
       │
       │ Security telemetry
       ▼
 Wazuh Manager
       │
       ├── Rule matching
       ├── Event analysis
       ├── Alert generation
       └── MITRE ATT&CK mapping
       │
       ▼
 Wazuh Dashboard
       │
       ▼
 SOC Investigation
```

---

## 6. Authentication Telemetry

Windows generates a large amount of authentication-related telemetry.

For example:

```text
Event ID 4624
```

represents a successful logon.

Other authentication events include:

```text
4625 — Failed logon
4648 — Explicit credential use
4672 — Special privileges assigned to new logon
4768 — Kerberos authentication ticket requested
4769 — Kerberos service ticket requested
4771 — Kerberos pre-authentication failed
```

These events must be analyzed in context rather than treated individually as evidence of malicious activity.

For example, repeated successful machine-account logons may represent normal Windows activity rather than an attack.

This is one of the reasons the project focuses on correlation and detection tuning rather than simply collecting large amounts of telemetry.

---

## 7. Security Boundaries

The laboratory is intentionally separated from the normal host network.

The SOC environment uses:

```text
10.10.10.0/24
```

with pfSense acting as the gateway.

The architecture allows controlled attack simulations to be performed against laboratory systems without intentionally targeting production systems or public infrastructure.

All offensive activity in the project is performed against systems owned and controlled within the laboratory.

---

## 8. Attack and Investigation Flow

Controlled attacks are introduced into the environment to generate realistic security telemetry.

The investigation workflow is:

```text
Controlled Attack
       │
       ▼
Endpoint / Network Activity
       │
       ▼
Telemetry Collection
       │
       ▼
Wazuh Processing
       │
       ▼
Detection / Alert
       │
       ▼
SOC Investigation
       │
       ▼
Timeline Reconstruction
       │
       ▼
Detection Improvement
```

---

## 9. Current Detection Architecture

The project is currently in the telemetry and investigation phase.

Current capabilities include:

* Windows authentication monitoring
* Sysmon telemetry collection
* Centralized Wazuh monitoring
* MITRE ATT&CK classification
* Endpoint event investigation
* Controlled attack simulation

Detection engineering is being developed incrementally.

The current workflow is:

```text
Understand telemetry
        ↓
Identify attack-related events
        ↓
Separate normal activity from suspicious activity
        ↓
Create detection logic
        ↓
Test against controlled attacks
        ↓
Tune false positives
        ↓
Document results
```

---

## 10. Future Architecture

The architecture will be expanded as the project develops.

Planned additions may include:

* Linux endpoints
* Network intrusion detection
* More detailed pfSense telemetry
* Centralized network flow monitoring
* Additional attack simulations
* Custom Wazuh rules
* Detection correlation
* Automated investigation workflows
* SOC dashboards
* Incident response playbooks
* Detection coverage tracking


This section will be updated as new components are introduced.

---

## 11. Related Documentation

* [Project README](../README.md)
* [SSH Brute-Force Investigation](../investigations/001-ssh-bruteforce.md)

