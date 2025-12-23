# Network Setup Context (OPNsense + VLANs + Switch + WireGuard)

## OPNsense Firewall
- Running on an Intel N100 box.
- VLANs configured:
  - VLAN 10 = LAN (10.0.10.0/24, gateway 10.0.10.1)
  - VLAN 20 = DMZ (10.0.20.0/24, gateway 10.0.20.1)
  - VLAN 30 = UNTRUSTED (10.0.30.0/24, gateway 10.0.30.1)
- IPv6 disabled on all interfaces (IPv6 Configuration = None).
- WAN is IPv4 DHCP, IPv6 disabled.
- DHCPv4 ISC server enabled for all VLANs.
- DNS = Unbound, with:
  - Register DHCP leases = enabled
  - Register static mappings = enabled
  - Domain = `home`
- `.home` DNS overrides exist for DMZ/LAN resources.
- WireGuard configured:
  - Server: 10.0.99.1/24
  - Clients get 10.0.99.x/32
  - Split-tunnel working correctly (AllowedIPs = VLAN10, VLAN20, WG subnet)
  - WAN rule correctly set for UDP/51820
  - WG interface assigned as `WG_VPN` with firewall rules:
    - Allow WG → LAN
    - Allow WG → DMZ
    - Allow WG → DNS (port 53)
    - Optional allow WG → any (internet passthrough)

## Switch Architecture (TP-Link SG108E)
- VLANs created:
  - VLAN 1 = Default (mgmt VLAN)
  - VLAN 10 = LAN
  - VLAN 20 = DMZ
  - VLAN 30 = UNTRUSTED
- IMPORTANT: TP-Link Easy Smart switches **always use VLAN 1 for management**, cannot be changed.
- Port roles:
  - Port 1 = Trunk to OPNsense (Tagged: 10,20,30; VLAN1 Tagged/Member)
  - Port 3 = **Trunk for Proxmox host** (Tagged: VLAN10 + VLAN20; VLAN1 Tagged/Member)
  - Port 7 = Trunk to EAP225 AP (Tagged: VLAN10 + VLAN30; VLAN1 Tagged/Member)
  - Port 8 = Trunk to another managed switch (Tagged: VLAN10 + VLAN30; VLAN1 Tagged/Member)
  - Port 2 = Access port for LAN devices (VLAN10 Untagged, PVID=10)
- Switch PVIDs:
  - Trunk ports (1,3,7,8) → PVID = 1
  - Access port (2) → PVID = 10
- iDRAC issue discovered:
  - iDRAC NIC was plugged into Port 3 (a trunk).
  - iDRAC does not support VLAN tags → could not obtain DHCP → autoconfig 169.254.x.x.
  - FIX: Move iDRAC to access port (Port 2), or convert its port to Untagged VLAN10 with PVID=10.

## AP Architecture (EAP225)
- AP supports VLAN tagging.
- Port 7 is a trunk (Tagged 10,30).
- EAP225 correctly configured with:
  - **Management VLAN = 10**
  - SSID1: VLAN 10 (LAN)
  - SSID2: VLAN 30 (UNTRUSTED)

## WireGuard Behavior
- Split tunnel confirmed:
  - Internal (`*.home`, LAN, DMZ) routes through WG.
  - Internet routes through client’s ISP.
- IPv4 traceroute on mobile shows 255.0.0.x hops due to carrier IPv6/464XLAT — normal behavior, not a WG issue.
- Internal route traceroute shows first hop 10.0.99.1 → correct.

## Untrusted VLAN DNS
- Untrusted VLAN set to use external DNS (1.1.1.1 / 8.8.8.8) via ISC DHCP to bypass adblock.
- Devices in VLAN30 will not resolve internal `.home` domains — expected behavior.

# Summary
This network is a multi-VLAN OPNsense deployment with WireGuard remote access, Omada EAP225 APs using VLANs for SSIDs, and TP-Link SG108E switches acting as VLAN distribution points. The main recent issue was iDRAC on a trunk port (fixed by assigning untagged VLAN10). WireGuard, DHCP, DNS, and VLAN routing are all functioning correctly.
