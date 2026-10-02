# OpenSOC — Mini Enterprise SOC Lab

A hands-on cybersecurity lab designed to simulate a small enterprise environment and practice Security Operations Center (SOC) workflows, including security monitoring, threat detection, incident investigation and detection engineering.

The project is built around a fully virtualized network — Active Directory, Windows endpoints, a pfSense gateway and a centralized Wazuh SIEM — running entirely on a single laptop under KVM/QEMU on Arch Linux.

## Objectives

* Build a realistic enterprise-style security monitoring environment
* Collect and analyze Windows security telemetry
* Monitor endpoint activity using Sysmon
* Centralize security events with Wazuh
* Simulate controlled attacks against the lab
* Investigate attacks using endpoint and network telemetry
* Develop and tune custom Wazuh detection rules
* Map detections to MITRE ATT&CK techniques
* Progressively improve detection quality and reduce false positives
* Document investigations and lessons learned

## Tech Stack & Infrastructure

| Layer                    | Tool                                      |
| ------------------------ | ----------------------------------------- |
| Hypervisor               | KVM/QEMU + libvirt (Arch Linux host)      |
| Gateway / Firewall / IDS | pfSense (Suricata package)                |
| Directory Services       | Windows Server 2022 (AD DS)               |
| Endpoint                 | Windows 11, Sysmon                        |
| SIEM                     | Wazuh Manager + Dashboard (Ubuntu Server) |

This lab intentionally avoids point-and-click hypervisors (VMware Workstation, VirtualBox) in favor of KVM/QEMU with manually defined libvirt networks — closer to how virtualization infrastructure is actually managed in production environments, and a deliberate constraint: the entire environment runs on a single laptop, so every architectural decision below accounts for that.

## Lab Architecture

```text
                         Internet
                            │
                       ┌────▼────┐
                       │ pfSense │
                       │ Gateway │  ← Suricata runs here
                       └────┬────┘
                            │
                     10.10.10.0/24
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
     Windows DC         Windows 11          Wazuh
     10.10.10.10        Endpoint            SIEM
          │                 │              10.10.10.20
        Sysmon            Sysmon              │
          │                 │                 │
      Wazuh Agent      Wazuh Agent            │
          │                 │                 │
          └─────────────────┴─────────────────┘
                            │
                       Wazuh Manager
                       Wazuh Dashboard
```

## Components

| Component           | Role                                          |
| ------------------- | --------------------------------------------- |
| pfSense             | Network gateway, firewall, and IDS (Suricata) |
| Windows Server 2022 | Active Directory domain controller            |
| Windows 11          | Domain client / endpoint                      |
| Ubuntu Server       | Wazuh SIEM                                    |
| Sysmon              | Windows endpoint telemetry                    |
| Wazuh Agent         | Endpoint log collection                       |
| Wazuh Manager       | Centralized security monitoring and detection |


**Wazuh over full Security Onion, for now.**
Security Onion's recommended resource footprint doesn't comfortably fit alongside a domain controller, an endpoint, and pfSense on one laptop. Wazuh's host-based agent model (lightweight footprint, logs forwarded directly from endpoints) gets the majority of the detection-engineering value — especially for AD/Windows telemetry — without requiring dedicated hardware. Network-layer IDS is covered separately via Suricata on pfSense above.

## Current Progress

### Infrastructure

* [x] pfSense gateway configured
* [x] Dedicated SOC network configured: `10.10.10.0/24`
* [x] Windows Server 2022 domain controller deployed
* [x] Windows 11 endpoint deployed
* [x] Ubuntu Server Wazuh instance deployed
* [x] Wazuh agents connected
* [x] Sysmon deployed on Windows systems

### Initial Security Testing

* [x] First controlled SSH brute-force simulation performed using Hydra
* [x] Windows authentication telemetry observed in Wazuh
* [x] Initial Wazuh event investigation started
* [ ] Investigate and document Sysmon process telemetry
* [ ] Build first custom Wazuh detection
* [ ] Develop brute-force correlation rule
* [ ] Create incident reports
* [ ] Expand attack scenarios
* [ ] Measure detection coverage and false positives

## First Investigation

The first controlled attack simulated an SSH brute-force attempt against the Windows domain controller.

The initial investigation revealed a significant amount of routine Windows authentication telemetry, including successful logon events such as Event ID 4624.

This led to the first detection-engineering objective of the project:

> Distinguish routine authentication telemetry from authentication activity that provides meaningful evidence of an attack.

Investigation details are documented in:

[`investigations/001-ssh-bruteforce.md`](investigations/001-ssh-bruteforce.md)

## Planned Detection Scenarios

The project will progressively investigate and detect scenarios such as:

* SSH brute-force authentication
* Suspicious process execution
* PowerShell activity
* Account abuse
* Credential-related activity
* Network reconnaissance
* Lateral movement
* Persistence techniques

Detection scenarios will be implemented incrementally as the lab develops.

## Philosophy

The goal of this project is not simply to deploy security tools.

The objective is to understand the complete SOC workflow:

```text
Attack
  ↓
Telemetry
  ↓
Detection
  ↓
Alert
  ↓
Investigation
  ↓
Correlation
  ↓
Incident Report
  ↓
Detection Improvement
```

All attacks are performed against isolated lab systems for educational and defensive security research purposes.

## Status

🚧 Active development

The project is currently in the initial telemetry and investigation phase.
