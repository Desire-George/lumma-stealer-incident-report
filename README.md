# 🚨 Incident Report: Lumma Stealer Fingerprinting & Payload Delivery
Pcap file - https://www.malware-traffic-analysis.net/2026/01/31/index.html

**Date:** 2026-01-27  
**Analyst:** Desire George  
**Severity:** 🔴 Critical  

## 📝 1. Executive Summary
Investigated a suspicious alert regarding **Victim Fingerprinting Activity** originating from the IP `153.92.1[.]49`. This activity is a known precursor to a Lumma Stealer infection, a Malware-as-a-Service (MaaS) threat where the malware gathers system specifications (OS, hardware, location) prior to initiating credential theft.

## 🔬 2. Technical Analysis

### Infection Vector & Payload Delivery
Using Wireshark, I isolated the web traffic with the following display filter:
`((http.request or tls.handshake.type eq 1) and !(ssdp))`

* **The Connection:** The victim machine (`10.1.21.58`) connected to `http://whitepepper[.]su` via TCP Port 80.
* **The Delivery:** By following the TCP Stream, I identified a Gzip-compressed payload being sent from the malicious server to the victim.

  <img width="3198" height="1401" alt="image" src="https://github.com/user-attachments/assets/43b2a11b-dfcf-44bf-bf73-aea3fc4509ee" />

### Threat Intelligence Validation
* **OSINT:** VirusTotal returned a **19/90 malicious detection rate** for the primary domain. 
* **Secondary C2 Infrastructure:** Further analysis identified two secondary Command & Control domains associated with this infection chain:
  * `holiday-forever[.]cc`
  * `communicationfirewall-security[.]cc`

<img width="3197" height="1400" alt="image" src="https://github.com/user-attachments/assets/0b754daa-7e37-48e6-a227-a0068825b726" />

### Victim Profiling
I pivoted to internal traffic to identify the scope of the threat and pinpoint the compromised identity.

* **IP / MAC Address:** `10.1.21.58` / `00:21:5d:c8:0e:f2`
* **Hostname:** `DESKTOP-ES9F3ML`
* **Compromised User:** Utilized the `kerberos.CNameString` filter and the "Find Packet" feature to successfully identify the user as **Gabriel Wyatt (`gwyatt`)**.

<img width="3198" height="1401" alt="image" src="https://github.com/user-attachments/assets/a5397579-30f0-43e8-a43e-1e9ba9974cfa" />
<img width="3198" height="1401" alt="image" src="https://github.com/user-attachments/assets/c9a66714-1a39-47e7-ba99-ea76393cef53" />


## 📰 3. Contextual Threat Intelligence
Lumma Stealer operates as a prominent Malware-as-a-Service (MaaS). Despite a massive global takedown in May 2025 that seized over 1,300 domains, the threat group re-emerged in July 2025 utilizing more covert delivery methods. The Gzip obfuscation identified in this capture is a direct reflection of these adapted, evasive tactics.

## 🛡️ 4. Conclusion & Recommended Actions
**Conclusion:** The host `DESKTOP-ES9F3ML` has been successfully fingerprinted by Lumma Stealer, and a compressed malicious payload was delivered. This constitutes a high-risk event for imminent credential theft and potential lateral movement.

**Response Playbook:**
1. **Containment:** Isolate `DESKTOP-ES9F3ML` from the network immediately to prevent data exfiltration and lateral movement.
2. **Eradication:** Wipe and re-image the compromised system to remove any hidden persistence mechanisms or secondary payloads.
3. **Prevention:** Block the `.su` and `.cc` Top-Level Domains (TLDs) at the perimeter firewall, assuming they are not required for legitimate business operations. Reset all credentials associated with the user `gwyatt`.
