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

For detailed information about each service, including access ports and functionality, see [Services.md](Services.md).

High-level service architecture:

    Media VM (VLAN 10)
    ├── Plex (Media Server)
    ├── *arr Stack (Radarr, Sonarr, Prowlarr, Bazarr)
    ├── Overseerr (Request Management)
    ├── qBittorrent (Download Client)
    ├── Gluetun (VPN Gateway)
    ├── Bookshelf (Ebook Library)
    ├── Watchtower (Auto-updater)
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

Most services use the Media VM's normal network stack (see [Services.md](Services.md) for complete list):
- Plex (Media Server)
- Radarr, Sonarr, Prowlarr, Bazarr (*arr Stack)
- Overseerr (Request Management)
- Bookshelf (Ebook Library)
- Watchtower (Auto-updater)

This ensures:
- Local LAN performance
- Reliable service discovery
- No VPN-related latency or breakage

---

## Service Communication

- *arr services (Radarr/Sonarr) communicate with qBittorrent via the Gluetun network namespace
- Prowlarr manages indexers for the *arr stack
- Overseerr integrates with Radarr/Sonarr for request handling
- Bazarr fetches subtitles based on *arr activity
- File access is handled through shared volumes
- Hardlinks / atomic moves are used where possible to avoid unnecessary disk I/O

For service-specific details and access information, refer to [Services.md](Services.md).

---

## Access Control

- Service web UIs are **LAN-only** (see [Services.md](Services.md) for ports)
- No direct exposure to VLAN 20
- Optional access via nginx, bound only to the VLAN 10 IP
- qBittorrent UI is not exposed via nginx for security

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
