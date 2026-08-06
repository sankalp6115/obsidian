This video provides a comprehensive, production-ready guide to building a **Site-to-Site VPN** using **Cloudflare Mesh**. The solution replaces traditional hub-and-spoke models (which often require public IPs and complex firewall rules) with an overlay network running on Cloudflare's global backbone (2:37 - 5:11).

### Core Architecture & Site Setup

- **Mechanism:** Every gateway establishes an encrypted, outbound connection to the nearest _Cloudflare Point of Presence (POP)_. Traffic is then routed across their private network to the destination (4:06 - 4:55).
- **Initialization:** Nodes are registered in the Cloudflare Zero Trust dashboard. The connector automatically handles tunnel interfaces, IP forwarding, and routing updates (9:10 - 10:09).
- **Split Tunneling:** Crucial for success, you must configure _Include_ lists in the mesh profile. This tells the gateway which local IP ranges (e.g., 10.1/24, 10.2/24) should be routed into the tunnel versus the local internet (13:02 - 14:47).

### High Availability (HA) & Redundancy

- **Active/Passive Replicas:** You can deploy multiple gateways (replicas) for a single site. Cloudflare handles inbound failover automatically, promoting a standby connector if the active one loses connectivity (17:29 - 18:04).
- **Keepalived for LAN Failover:** To ensure local clients don't lose their gateway, the video suggests using _Keepalived_. This creates a **floating virtual IP** (VIP) shared between two gateways. A health-check script monitors the tunnel status; if the primary fails, the VIP migrates to the secondary gateway in seconds (21:29 - 25:52).
- **Symmetric Routing:** For TCP connections, it is critical that inbound and outbound traffic use the same connector to avoid packet loss caused by spoofing detection (19:15 - 20:30).

### Advanced Configurations

- **MikroTik Integration:** Since RouterOS v7 supports native _WireGuard_, you can bridge it to the mesh. Because modern connectors default to the _MASQUE_ protocol, you must change the device profile in Cloudflare to _WireGuard_ to ensure compatibility (27:42 - 28:34).
- **Remote Users (Road Warriors):** Using the Cloudflare One agent, remote laptops can join the mesh. Enrollment is managed via identity providers (e.g., One-time PIN). A key step is adding a static route on internal clients so they know how to send return traffic to the road warrior's _100.96.x.x_ mesh IP (32:27 - 37:45).
- **Centralized Firewall Policies:** Instead of managing local device firewalls, you can enforce **Layer 4 policies** (allow/block by IP or User Identity) directly at the Cloudflare edge. Note: These policies typically inspect TCP/UDP, so they may not block _ICMP (ping)_ traffic (39:06 - 41:50).

