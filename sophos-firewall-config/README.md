# Sophos XG Firewall Configuration

## Objective
Configure Sophos XG Firewall in a VMware virtual lab and implement three security policies to enforce granular controls, web filtering, application control, and firewall rules, for client VMs on a Host-Only network.

## Tools & Technologies
- Sophos XG Firewall
- VMware Workstation
- Kali Linux (for testing/verification)

## What I Did

**Initial Network Interface Configuration**
- Configured the LAN interface (Port1) with a static IP (`192.168.56.72/24`) as the internal network gateway
- Configured the WAN interface (Port2) via DHCP to connect the firewall to the external network

**Policy 1 — Allow_LAN_to_Internet (Fallback Rule)**
- Created a general allow rule (LAN → WAN, any service) with no Web Filter or App Control attached, to serve as the lowest-priority fallback rule

**Policy 2 — Block Social Media / Gambling / Adult Content**
- Built a Web Policy blocking the categories Games and Gambling, Nudity and Adult Content, and Social Networking
- Created a firewall rule (LAN → WAN, HTTP/HTTPS) with the Web Filter attached, placed above the fallback rule so it takes priority

**Policy 3 — Block P2P Apps (Application Control)**
- Applied Sophos's predetermined "Block peer to peer (P2P) networking apps" policy via Application Control
- Created a firewall rule (LAN → Any, all services) to allow deep packet inspection across all ports, placed second in rule priority, below Policy 2

## Testing & Verification (performed on Kali Linux)
- Confirmed default gateway and first hop via `ip route show default` and `traceroute -n 8.8.8.8`, verifying traffic routed through the Sophos firewall
- Attempted to visit blocked sites: a gambling site and Facebook both returned blocked, confirming the web filtering policy worked as intended

## Screenshots
<img width="600" src="PASTE_URL_HERE" />

*Fig 1: Network topology*

<img width="600" src="PASTE_URL_HERE" />

*Fig 2: LAN interface (Port1) configuration*

<img width="600" src="PASTE_URL_HERE" />

*Fig 3: WAN interface (Port2) configuration*

<img width="600" src="PASTE_URL_HERE" />

*Fig 4: Allow_LAN_to_Internet fallback rule*

<img width="600" src="PASTE_URL_HERE" />

*Fig 5: Web policy blocking social media, gambling, and adult content*

<img width="600" src="PASTE_URL_HERE" />

*Fig 6: Firewall rule enforcing Policy 2*

<img width="600" src="PASTE_URL_HERE" />

*Fig 7: Firewall rule enforcing Policy 3 (P2P blocking)*

<img width="600" src="PASTE_URL_HERE" />

*Fig 8: Full rule list showing correct priority order*

## Key Takeaways
- Firewall rule order is critical in Sophos XG, blocking/filtering rules must sit above general allow rules or they'll never be evaluated
- Web Filter and Application Control are attached at the rule level, not standalone, so each policy needs both the underlying filter/control policy and a firewall rule referencing it
- Verifying from an external testing machine (Kali) rather than trusting the config alone confirmed the policies were actually enforced, not just configured
