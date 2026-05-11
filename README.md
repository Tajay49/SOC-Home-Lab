# SOC Home Lab – Splunk & Sysmon Threat Detection #

## Overview
This project demonstrates a mini Security Operations Center (SOC) home lab built using Splunk Enterprise, Sysmon, Kali Linux, and VirtualBox to simulate real-world attack and detection scenarios.

The objective of this lab was to gain hands-on experience in:
- SIEM configuration and log ingestion
- Endpoint monitoring and telemetry collection
- Threat detection and investigation
- SPL querying and event analysis
- Attack simulation and IOC identification

The environment simulates a real SOC workflow where logs are generated on a Windows endpoint, forwarded into Splunk, and analyzed for suspicious activity.

---

# Lab Architecture

## Environment
- Windows 11 Endpoint
- Kali Linux Attacker Machine
- Splunk Enterprise SIEM
- Sysmon
- Splunk Universal Forwarder
- Oracle VirtualBox

## Architecture Flow
Kali Linux (Attacker) → Windows 11 Endpoint → Sysmon → Splunk Universal Forwarder → Splunk Enterprise

---

# Screenshots

## SOC Home Lab Setup
![SOC Lab](screenshots/lab-setup.png)

## Kali Linux Connectivity Test
![Kali Ping](screenshots/kali-ping.png)

## Splunk Log Ingestion
![Splunk Logs](screenshots/splunk-logs.png)

## Lab Architecture Diagram
![Architecture](screenshots/lab-architecture.png)

---

# Log Sources Monitored

The following logs were ingested into Splunk:

- Sysmon Operational Logs
- Windows Defender Logs
- PowerShell Operational Logs
- Windows Security Logs
- Windows System Logs
- Windows Application Logs

---

# Splunk Configuration

Example `inputs.conf` configuration:

```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

Attack Simulation

The lab included simulated malicious activity from the Kali Linux machine to generate realistic security events.

Activities included:

Network connectivity testing using ICMP (ping)
Suspicious file transfer and execution
Reverse shell activity
Process execution monitoring
Network connection analysis
Detection & Investigation

Using Splunk Processing Language (SPL), logs were analyzed to identify indicators of compromise (IOCs) and suspicious behavior.

Example searches:

index=endpoint EventCode=1
index=endpoint powershell
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"

Investigation activities included:

Process chain analysis
Suspicious parent-child relationship identification
IOC detection
Timeline reconstruction
Endpoint activity monitoring
Troubleshooting & Recovery

During setup, endpoint log ingestion issues were encountered within Splunk. The environment was restored by reverting the virtual machine to a previous stable snapshot and reconfiguring the Splunk environment. This restored endpoint visibility and allowed attack simulations and detections to continue successfully.

Skills Demonstrated
SIEM Operations (Splunk)
Threat Detection & Threat Hunting
SPL Querying
Endpoint Monitoring with Sysmon
Incident Investigation
Log Analysis & Event Correlation
Virtualization (VirtualBox)
Network Monitoring & Analysis
Future Improvements

Planned enhancements for the lab include:

Active Directory integration
Custom Splunk dashboards
Advanced detection rules
MITRE ATT&CK mapping
Automated alerting
Additional attack simulations
Tools & Technologies
Splunk Enterprise
Sysmon
Kali Linux
Windows 11
Splunk Universal Forwarder
Oracle VirtualBox
Disclaimer

This project was created for educational and defensive cybersecurity purposes only within an isolated lab environment.

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false
