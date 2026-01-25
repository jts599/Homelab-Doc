# Media Stack Architecture

## Overview
All media-related services run inside a single **Media VM** on VLAN 10.  
Services are separated by port and container boundaries rather than by VM.

This approach balances:
- Simplicity
- Resource efficiency
- Operational clarity

---

## Media VM Responsibilities

The Media VM hosts:
- Media server
- Media automation services
- Download client
- VPN egress for downloads

The VM itself:
- Has **only a VLAN 10 interface**
- Is not directly reachable from the DMZ
- Is accessed via nginx or local/VPN access only

---

## Containerization Strategy

Docker is used inside the Media VM to isolate services.

High-level layout:

    Media VM (VLAN 10)
    ├── Plex
    ├── Radarr
    ├── Sonarr
    ├── qBittorrent
    ├── Gluetun (VPN container)
    └── Shared media volumes

---

## VPN + Download Isolation

### Gluetun
- Acts as a dedicated VPN gateway
- Provides a kill-switch and namespace isolation
- Handles all outbound torrent traffic

### qBittorrent
- Runs in Docker with its network namespace attached to Gluetun
- Uses Docker configuration equivalent to:

      network_mode: service:gluetun

- Has **no independent network access**
- Cannot leak traffic outside the VPN

Only qBittorrent is routed through the VPN.

---

## Non-VPN Services

The following services use the Media VM’s normal network stack:
- Plex
- Radarr
- Sonarr
- Any other media management tools

This ensures:
- Local LAN performance
- Reliable service discovery
- No VPN-related latency or breakage

---

## Service Communication

- Radarr/Sonarr communicate with qBittorrent via the Gluetun network namespace
- File access is handled through shared volumes
- Hardlinks / atomic moves are used where possible to avoid unnecessary disk I/O

---

## Access Control

- Service web UIs are **LAN-only**
- No direct exposure to VLAN 20
- Optional access via nginx, bound only to the VLAN 10 IP
- qBittorrent UI is not exposed via nginx

---

## Failure Domains

This architecture intentionally accepts:
- A single VM as a failure domain
- Shared lifecycle for media-related services

Tradeoffs:
- Simpler backups
- Easier storage management
- Lower memory overhead

Isolation is achieved where it matters most: network egress and exposure.

---

## Rationale

This setup:
- Prevents torrent traffic leaks
- Keeps VPN complexity contained
- Avoids routing the entire VM through a VPN
- Makes access rules explicit and enforceable at the network layer

---

## Future Considerations

- Separate VM if trust levels diverge
- Read-only bind mounts for Plex
- Automated container updates
- Monitoring VPN health and torrent connectivity
