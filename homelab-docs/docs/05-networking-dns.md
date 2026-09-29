# Networking & DNS: Pi-hole

## Purpose

Network-level ad and tracker blocking via DNS sinkholing, deployed as a
containerized recursive resolver rather than installed on bare metal.

## Deployment

```bash
docker run -d --name pihole \
  -p 53:53/tcp -p 53:53/udp \
  -p 8080:80 \
  -e TZ="<your-timezone>" \
  -e FTLCONF_dns_listeningMode="all" \
  -v ~/pihole/etc-pihole:/etc/pihole \
  -v ~/pihole/etc-dnsmasq.d:/etc/dnsmasq.d \
  --restart unless-stopped \
  --cap-add NET_ADMIN \
  pihole/pihole
```

Two non-obvious requirements surfaced during deployment and are called
out explicitly here since they're easy to miss:

### 1. Port 53 conflicts with the host's own DNS stub resolver

Modern Ubuntu/Mint systems run `systemd-resolved`, which binds port 53
on `127.0.0.53`/`127.0.0.54` by default, conflicting with any container
trying to publish port 53 to the host. Resolved by disabling
`systemd-resolved`'s local stub listener and pointing the host's own
resolution at the containerized Pi-hole instead:

```bash
# /etc/systemd/resolved.conf
DNSStubListener=no
```

```bash
sudo rm /etc/resolv.conf
sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolv.conf'
sudo systemctl restart systemd-resolved
```

### 2. Default listening mode rejects LAN queries

By default, Pi-hole's `dnsmasq` backend only answers queries it
considers "local" to its own container network, silently rejecting
queries from the actual LAN with a log line like
`dnsmasq: ignoring query from non-local network <ip>`. This is
resolved with `FTLCONF_dns_listeningMode="all"` (included in the run
command above), which is the documented fix for Pi-hole-in-Docker
deployments answering queries from outside the container's own subnet.

## Network-wide enforcement limitation

The consumer-grade router in this environment (a Cox Panoramic Wifi
gateway) does not expose a DNS override setting under any menu — Port
Forwarding, Port Triggering, Remote Management, DMZ, and Device
Discovery are the only options under "Advanced." As a result, DNS
filtering here is enforced **per device** (manually pointing each
device's DNS at the Pi-hole host) rather than automatically for every
device on the network via DHCP. A documented limitation of consumer ISP
hardware, and the kind of constraint a production network would resolve
with a router that exposes DHCP option 6 configuration.

## Known scope limitation

DNS-level blocking cannot distinguish ad traffic from content traffic on
platforms that serve both from the same domain (notably YouTube and
Hulu). This is a structural limitation of DNS-based filtering, not a
misconfiguration, and is worth stating plainly rather than overselling
what this component covers.
