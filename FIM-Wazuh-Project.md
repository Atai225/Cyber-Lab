# Endpoint Security: Implementing File Integrity Monitoring (FIM) via Wazuh XDR

## 🎯 Project Overview
This project demonstrates the deployment, configuration, and validation of an enterprise-grade **File Integrity Monitoring (FIM)** solution using the **Wazuh XDR/SIEM** platform. The primary objective was to secure a critical directory on a Windows 7 endpoint, establish a secure baseline, monitor unauthorized modifications in real-time, and analyze ingested telemetry from a SOC Analyst's perspective.

### Demonstrated Skills & Concepts:
* **SIEM/XDR Architecture:** Agent-Manager communication, log collection, and alert generation.
* **Detection Engineering:** Configuring endpoint policies (`ossec.conf`) for real-time telemetry ingestion.
* **Log Analysis & Incident Response:** Inspecting raw JSON payloads, identifying triggered rule IDs, and analyzing cryptographic hash variations (MD5/SHA256).
* **Infrastructure Management:** Linux-based SIEM administration (Ubuntu Server) under constrained resource environments utilizing swapping techniques.

---

## 🛠️ System Architecture & Lab Topology

The laboratory environment is fully isolated inside an internal virtual network using Oracle VirtualBox (**NAT Network** topology), preventing external exposure while maintaining full local communication.

* **SIEM / Manager Node:** Ubuntu Server 22.04 LTS (All-in-one Wazuh Manager installation) | IP: `10.0.2.15`
* **Monitored Endpoint:** Windows 7 Ultimate (Wazuh Agent v4.7.5) | IP: `10.0.2.4`

<img width="1917" height="991" alt="image" src="https://github.com/user-attachments/assets/50f34d9b-04f3-4dae-9336-c27172bdf93c" />


---

## 🚀 Step-by-Step Implementation

### Phase 1: Endpoint Configuration & Real-Time Policy Deployment
To capture unauthorized modifications instantly rather than waiting for scheduled interval scans, the Wazuh agent's core configuration was modified to monitor the target directory `C:\Lab-FIM` in **real-time mode**.

1. A secure directory was provisioned on the asset: `C:\Lab-FIM`.
2. The endpoint's operational configuration file (`ossec.conf`) was updated under administrative privileges to include the following cryptographic monitoring parameters:

<syscheck>
  <directories realtime="yes" check_all="yes">C:\Lab-FIM</directories>
</syscheck>

* **Defensive Rationale:** * `realtime="yes"` enables instant event-driven hooks within the OS filesystem.
  * `check_all="yes"` enforces the inspection of file size, permissions, owner, and cryptographic hashes (MD5, SHA1, SHA256).

3. The `WazuhSvc` daemon was restarted via PowerShell to flush and apply the new detection policy.


<img width="1022" height="767" alt="Снимок экрана 2026-05-16 231033" src="https://github.com/user-attachments/assets/3a047b48-e710-4fca-93a3-c07a5022168b" />


---

### Phase 2: Simulating Malicious Activity (Red Team Simulation)

To validate the detection capabilities, a multi-stage data modification attack was simulated inside the `C:\Lab-FIM` directory:

* **Data Creation:** A file named `credentials.txt` containing dummy administrative credentials was created.
* **Data Tampering (Modification):** The contents of `credentials.txt` were altered to simulate an unauthorized privilege escalation or data corruption attempt.
* **Defense Evasion (Deletion):** The file was deleted to simulate log wiping or destruction of evidence by the adversary.

---

## 📊 Telemetry Analysis & SIEM Verification (Blue Team Investigation)

Upon reviewing the Wazuh Indexer Dashboard within the **Integrity Monitoring** module, the system successfully parsed and flagged all simulated adversary actions.

The security events were mapped to specific **Wazuh Rule IDs** and detailed system actions:


<img width="1857" height="887" alt="image" src="https://github.com/user-attachments/assets/ea2c2205-8afa-4b1d-9726-36eca097fcdb" />



### Evidence 1: Complete Attack Lifecycle Timeline
The XDR dashboard accurately correlates the events chronologically, proving continuous monitoring integrity.

<img width="1853" height="887" alt="image" src="https://github.com/user-attachments/assets/07ae4325-e67d-447b-be85-5d2bd03208b2" />


### Evidence 2: Cryptographic Hash Variance Analysis
By expanding the metadata of the **Rule 550 (File modified)** event payload, we can extract the precise forensic footprint left by the modification. The SIEM captured the exact state change by comparing pre- and post-compromise cryptographic signatures:

* **MD5 Before:** `d41d8cd98f00b204e9800998ecf8427e`
* **MD5 After:** `d3f43d15a415050d4aaeccf1fba4e765`
* **User responsible:** `vboxuser`

<img width="647" height="756" alt="image" src="https://github.com/user-attachments/assets/6bc6b5c1-5994-4786-8b17-ab416f9ce529" />

<img width="873" height="755" alt="image" src="https://github.com/user-attachments/assets/1c5ca3e8-0fb6-4d4c-8441-3e76f596d8a1" />

---

## 🧠 Strategic Insights & Remediation

### Security Value
Implementing FIM provides defensive teams with an early indicator of compromise (IoC). In an enterprise scenario, this setup effectively neutralizes:
* **Ransomware Attacks:** Rapid detection of bulk file modifications/encryption.
* **System Tampering:** Unauthorized modification of critical binaries (e.g., `C:\Windows\System32\drivers\etc\hosts` or system DLLs).

### Next Steps for Enterprise Hardening
While monitoring is effective, a true SOC workflow requires automated response. The next iteration of this project will involve configuring **Wazuh Active Response** policies to trigger an automated PowerShell script that isolates the endpoint network interface or blocks the offending user account immediately upon a high-severity FIM alert trigger.
