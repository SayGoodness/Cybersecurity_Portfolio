# Secure Security Architecture — PayLink Africa (Fintech)

## Objective
Design a three-tier security architecture for a fictional high-growth fintech (PayLink Africa) handling PII and payment transaction data, applying the CIA Triad, Defense in Depth, and Least Privilege principles, aligned to NIST SP 800-53, PCI-DSS, ISO/IEC 27001, and OWASP Top 10.

## Frameworks Referenced
- NIST SP 800-53 / NIST Cybersecurity Framework (CSF)
- PCI-DSS
- ISO/IEC 27001
- OWASP Top 10
- SABSA model

## Architecture Overview
A three-tier design (DMZ / application / database) built to protect financial data flows end to end.

## Design Highlights

**Confidentiality**
- All client traffic secured via TLS 1.3
- Data at rest protected with AES-256 encryption
- Database isolated behind internal firewalls via VPC segmentation

**Integrity**
- SHA-256 hashing on all bank API transactions, mismatches trigger automatic rejection
- Web Application Firewall + parameterized queries to block SQL injection
- WORM logs and centralized SIEM logging for an immutable audit trail

**Availability**
- External load balancer distributing traffic across redundant web servers
- Cloud-based DDoS scrubbing at the network edge
- 3-2-1 backup rule using immutable backups for rapid ransomware/hardware-failure recovery

**Defense in Depth & Least Privilege**
- Layered perimeter defense: Next-Gen Firewall → WAF → IDS/IPS → EDR on each host → encryption as a final fail-safe
- Role-Based Access Control: web servers get read-only database access, blocked from direct DB communication by internal firewalls
- Just-In-Time access via a secure jump host for admin-level changes, requiring an encrypted VPN tunnel

## Threat Mitigation
- **SQL Injection:** blocked by WAF + parameterized queries
- **DDoS:** mitigated via load balancer + scrubbing services
- **Ransomware:** immutable, cryptographically locked backups ensure recovery without paying ransom

## Architecture Design
<img width="600" src="https://github.com/user-attachments/assets/9d08ee70-92fe-4c5b-af8f-5d0431643e29" />


*Fig 1: Three-tier architecture diagram*

## Key Takeaways
- The hardest design tradeoff was balancing security depth against transaction latency, every added inspection layer (WAF, deep packet inspection, encryption) costs time, and fintech customers expect instant confirmations, solved by placing load-balancing strategically to absorb that cost
- In a fully cloud-native version of this architecture, physical firewalls and IDS/IPS would be replaced by Security Groups, NACLs, and cloud-native detection (e.g. AWS GuardDuty), with the Shared Responsibility Model shifting physical security to the cloud provider
- Security is a diminishing-returns exercise: the goal isn't buying every available tool, it's applying Least Privilege and Defense in Depth precisely, hardening the "crown jewels" (database) while keeping the public-facing layer fast and usable
