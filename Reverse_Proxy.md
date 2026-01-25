# Reverse Proxy Architecture

## Overview
This homelab uses a centralized Nginx reverse proxy to control access to all HTTP-based services.  
The proxy is responsible for:
- Acting as the single ingress point for user-facing services
- Enforcing VLAN-based access boundaries
- Preventing direct access to application ports from untrusted networks

The design prioritizes:
- Clear trust separation
- Minimal exposed surface area
- Simple, auditable routing rules

---

## Network Segmentation

### VLANs
- **VLAN 10 – Local**
  - Trusted internal network
  - All application services live here
  - Direct access allowed only from LAN or VPN

- **VLAN 20 – DMZ**
  - Untrusted / semi-trusted network
  - Only the reverse proxy is reachable
  - No direct access to backend services

---

## Reverse Proxy Placement

- **Nginx runs in its own LXC/container**
- The proxy has **two network interfaces**:
  - One IP in VLAN 10
  - One IP in VLAN 20

This allows nginx to:
- Accept connections from both networks
- Decide which services are reachable from which VLAN
- Proxy traffic into VLAN 10 without exposing backend ports

---

## Access Model

### Public / DMZ-Accessible
- **Public website**
  - Exposed via nginx listening on the VLAN 20 IP
  - Backend service remains in VLAN 10
  - No other services are reachable from VLAN 20

### Local-Only Services
- Plex
- Media management applications (Radarr, Sonarr, etc.)
- Internal tools

These services:
- Are **not exposed in the DMZ**
- Are accessible only via:
  - VLAN 10 (local LAN)
  - OPNsense-hosted VPN (which terminates into VLAN 10)

---

## Plex Access Strategy

Plex is intentionally **not public-facing**.

Access methods:
- Local LAN (VLAN 10)
- VPN into OPNsense → VLAN 10

Benefits:
- No open ports or public endpoints
- Avoids relying on Plex remote access
- Keeps media traffic off the DMZ entirely

---

## Firewall Enforcement

Firewall rules enforce the proxy model:

- VLAN 20 → Backend service ports: **DENY**
- VLAN 20 → Nginx: **ALLOW**
- VLAN 10 → Backend services: **ALLOW**
- VPN → VLAN 10: **ALLOW**

Nginx is the only system permitted to bridge traffic from DMZ to LAN.

---

## Rationale

This design:
- Prevents accidental exposure of internal services
- Keeps application complexity out of the DMZ
- Allows services to be added or removed without firewall changes
- Makes ingress behavior explicit and reviewable in nginx config

---

## Future Considerations

- TLS termination at nginx
- Optional authentication for select internal services
- Per-service rate limiting if public exposure increases
