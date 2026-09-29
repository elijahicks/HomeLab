# Architecture

## Hardware

Retired Intel MacBook Pro (no Apple T2 security chip), internal drive
only at time of writing. Chosen deliberately to demonstrate getting
production-grade services running on constrained, non-purpose-built
hardware rather than a purchased server or Raspberry Pi.

## Design decisions

**Docker over bare-metal installs.** Every service (Jellyfin, Pi-hole,
Portainer) runs as an isolated container rather than installed directly
on the host. This keeps dependencies contained, makes each service
independently disposable/rebuildable, and mirrors how services are
deployed in production environments.

**Tailscale over traditional port forwarding.** Rather than opening
inbound ports on the home router (a larger attack surface, and not
possible at all here since the ISP-provided gateway doesn't expose the
needed controls), remote access runs over a WireGuard-based mesh
network. Devices that can't run a VPN client (smart TVs) are handled
separately via Tailscale Funnel, which exposes only the single service
needed, over HTTPS, rather than opening the network itself.

**Pi-hole as a DNS sinkhole, not a network-wide policy.** Ideally, DNS
filtering is enforced at the router/DHCP level for every device
automatically. The consumer router in this environment doesn't expose
a DNS override option, so DNS is configured per-device instead. This is
a real-world constraint documented explicitly here rather than glossed
over, along with what a fuller deployment (a pfSense/OPNsense router,
or a router with custom firmware) would change.

## Data flow summary

1. **Local playback**: a LAN client requests `http://<static-ip>:8096`
   directly; Jellyfin serves the stream from its bind-mounted media
   folder.
2. **Remote playback (Tailscale-capable device)**: the device joins the
   same tailnet, and reaches Jellyfin over its Tailscale IP
   (`100.x.x.x`) as if it were on the LAN, over an encrypted tunnel.
3. **Remote playback (Roku / non-VPN device)**: Tailscale Funnel
   exposes Jellyfin's port at a public HTTPS URL
   (`https://<host>.<tailnet>.ts.net`); the device connects directly
   to that URL with no VPN client involved.
4. **DNS filtering**: any device with its DNS manually set to the
   host's static IP sends queries to Pi-hole, which filters against
   blocklists before forwarding legitimate queries upstream.
