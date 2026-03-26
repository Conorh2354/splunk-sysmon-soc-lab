# Splunk + Sysmon SOC Home Lab

## Objective
Build a small SOC-style home lab to practice endpoint telemetry, log ingestion, and security event analysis using Splunk and Sysmon on Windows 11. 

## Lab Architecture
- Hypervisor: VirtualBox on Windows 11 host
- VMs:
  - Windows 11 "Victim" endpoint with Sysmon installed
  - Kali VM for future attack simulation
- SIEM: Splunk Enterprise Free running on Windows 11 VM
- Logging:
  - Sysmon operational logs forwarded to Splunk
  - Windows Security / System Event Logs forwarded to Splunk

## Tools Used
- Splunk Enterprise (Free license) – log ingestion, queries, dashboards 
- Sysmon – process, network, file and registry telemetry on Windows 
- VirtualBox – virtualization platform
- Windows 11 – endpoint(s) and Splunk server OS

## Configuration Steps (High-Level)
1. Created Windows 11 VM(s) following MyDFIR home lab series.
2. Installed Splunk Enterprise and configured a basic index and input for Windows logs.
3. Installed Sysmon with a community configuration.
4. Confirmed Sysmon events (Event ID 1, 3, etc.) were ingested into Splunk via simple SPL searches.
5. Built a basic Splunk dashboard to monitor process creation and logon activity. 

## Skills Demonstrated
- Building and managing a small virtualized SOC lab
- Installing and configuring Splunk for endpoint log ingestion
- Deploying Sysmon for high-fidelity Windows telemetry
- Writing basic SPL searches to investigate security events
- Documenting technical lab work for a security portfolio 
