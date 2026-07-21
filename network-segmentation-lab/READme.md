# Network Segmentation Lab (Cisco Packet Tracer)

## Objective
Build a small, secure enterprise network in Cisco Packet Tracer for three departments (Admin, Sales, IT), each on its own VLAN and subnet. Implement VLAN segmentation, inter-VLAN routing via Router-on-a-Stick, and ACLs to restrict cross-department access according to a least-privilege policy, secured with SSH-only remote management.

## Tools & Technologies
- Cisco Packet Tracer
- VMware Workstation
- Command Line Interface (CLI)

## Network Topology
1 Router (2911), 1 Switch (2960), 3 end devices representing Admin, Sales, and IT, connected via copper straight-through cabling.

## What I Did

**VLAN Segmentation**
- Created three VLANs on the switch (e.g. VLAN 10 for Admin) and assigned switch ports to access mode per department
- Configured the switch-to-router link as a trunk port (`switchport mode trunk`) to carry traffic from all three VLANs over a single physical link

**Inter-VLAN Routing (Router-on-a-Stick)**
- Created three sub-interfaces on the router's `GigabitEthernet0/0` port, one per VLAN, using `encapsulation dot1Q` to tag traffic
- Assigned each sub-interface a gateway IP, turning the router into the Layer 3 point of control between VLANs

**Access Control Lists (ACLs)**
- Built Extended ACLs to enforce department access policy: Admin → Sales only, Sales → IT only, IT unrestricted
- Ordered rules carefully (permit before deny, explicit `permit ip any any` fallback), since ACLs are evaluated sequentially
- Debugged a misapplied ACL name (`ACL_ADMIN` vs. the correct `ip access-group ADMIN_ACL`) using `show access-lists`, corrected it, and re-verified with ping tests

**SSH Configuration (Secure Remote Access)**
- Configured hostname, domain name, and generated 2048-bit RSA keys on router and switch to enable SSH
- Created a local administrative account with a dedicated encrypted secret (`privilege 15`), avoiding reliance on a shared enable password
- Locked down VTY lines to `transport input ssh` and `login local`, disabling Telnet entirely

## Security Justification
- Applied the Principle of Least Privilege (PoLP): Admin restricted from IT's sensitive resources, Sales limited to IT support access only, IT retains full access for maintenance
- Controlling inter-VLAN communication reduces the internal attack surface and limits unauthorized lateral movement

## Testing & Verification
- Verified VLAN assignments with `show vlan brief`
- Confirmed inter-VLAN routing worked (pre-ACL) via cross-department pings
- Verified sub-interfaces with `show ip interface brief`
- Confirmed SSH access worked (`ssh -l admin 192.168.10.1`) and Telnet access was correctly denied
- Tested ACL enforcement: Admin → Sales ping failed as expected

## Screenshots
<img width="600" src="https://github.com/user-attachments/assets/30c98a30-f966-4b74-a286-b9d27b946e68" />

*Fig 1: VLAN configuration on the switch*


<img width="600" src="https://github.com/user-attachments/assets/243f099b-8fd0-4ac8-aeec-feadb02e54e2" />

*Fig 2: Router sub-interface configuration (Router-on-a-Stick)*


<img width="600" src="https://github.com/user-attachments/assets/09c321ba-1c04-4af7-bbb6-6092dc809b4b" />
<img width="600" src="https://github.com/user-attachments/assets/a809a712-441e-4622-bb0d-9e679a71448b" />

*Fig 3: ACL configuration and verification*


<img width="600" src="https://github.com/user-attachments/assets/cd7d01fb-9b27-4958-af0d-9f957e3bd91b" />
<img width="600" src="https://github.com/user-attachments/assets/74e58bfa-2ac0-4592-a85f-50dd3f3dd218" />

*Fig 4: SSH connection test confirming secure access*

## Key Takeaways
- ACL order and placement are critical, a misplaced or incorrectly sequenced rule can silently fail to enforce security or accidentally break legitimate connectivity
- Router-on-a-Stick is an efficient way to route between VLANs using a single physical interface, but requires precise sub-interface and trunk configuration
- Enforcing SSH-only access and disabling Telnet is a simple but essential step to prevent plaintext credential capture
