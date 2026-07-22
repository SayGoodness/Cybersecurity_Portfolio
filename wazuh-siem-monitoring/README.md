# Wazuh SIEM & Endpoint Monitoring

## Objective
Deploy a Wazuh Manager in a virtualized environment, connect a Windows endpoint via the Wazuh Agent, and configure File Integrity Monitoring (FIM) and compliance dashboards to demonstrate centralized security monitoring and threat detection.

## Tools & Technologies
- Wazuh (Manager, Indexer, Dashboard)
- VMware Workstation Pro
- Windows endpoint with Wazuh Agent

## What I Did

**Network & Deployment**
- Configured VMnet8 (NAT) in VMware's Virtual Network Editor, allowing lab VMs to reach the internet for updates while staying isolated from the physical network
- Deployed the Wazuh Server using the pre-built OVA image, allocating 4GB RAM and 2 CPU cores
- Verified the server's DHCP-assigned IP and accessed the Wazuh Dashboard over HTTPS (self-signed cert)

**File Integrity Monitoring (FIM)**
- Created a monitored test directory and file on the Windows endpoint
- Edited the agent's `ossec.conf` to add the target directory under the `<syscheck>` block with `check_all`, `report_changes`, and `realtime` enabled
- Restarted the Wazuh agent service to apply the configuration

**Verification**
- Confirmed the monitored file appeared in the Dashboard's FIM Inventory
- Modified the file and confirmed an "Integrity checksum changed" event appeared, cross-checked via rule ID 550 in the Events tab
- Deleted the file and confirmed it dropped from inventory with a "deleted" status, cross-checked via rule ID 553

**Compliance Mapping**
- Reviewed the PCI DSS dashboard, mapping alerts to compliance controls relevant to payment card data handling
- Reviewed the GDPR dashboard, mapping security events to relevant articles for data protection/privacy risk visibility

## Screenshots
<img width="600" src="https://github.com/user-attachments/assets/8c60d830-3217-47b1-aba8-eb7cdfb53fa0" />

*Fig 1: Wazuh Dashboard overview*

<img width="600" src="https://github.com/user-attachments/assets/3e6d01b7-9925-4ea5-86ec-6938e7b4bfaf" />

*Fig 2: ossec.conf syscheck configuration*

<img width="600" src="https://github.com/user-attachments/assets/db290b29-d490-4a3b-beeb-8a2a33e50584" />

*Fig 3: FIM alert confirming file modification detected*

<img width="600" src="https://github.com/user-attachments/assets/bf4054ae-4bf2-4acc-b037-e6622cc53485" />

*Fig 4: PCI DSS compliance dashboard*

## Key Takeaways
- FIM alerts required correct `ossec.conf` indentation, a small formatting mistake was enough to silently break detection, reinforcing the value of always verifying with a controlled test (create → modify → delete)
- Wazuh's native compliance dashboards (PCI DSS, GDPR) turn raw security alerts into audit-ready evidence, useful beyond just technical detection
