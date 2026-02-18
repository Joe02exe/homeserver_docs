# Raspberry Pi Home Server Setup

This repository documents how I built and evolved my personal home server - from a manual OpenMediaVault + Docker setup to a clean Umbrel-based system.

The project helped me deeply understand networking, containerization, storage management, permissions, and self-hosted infrastructure.

## Hardware
- Geekworm NASPi Gemini 2.5 Dual SATA NAS Kit ([link](https://geekworm.com/products/naspi-gemini))
- 2x Samsung 4TB SSDs
- Fully cable-connected network (no WiFi)
- No SD card: entire system runs from SSDs (better performance & reliability)

The NASPi case supports hardware RAID and includes configurable pins for:
- Auto fan
- Auto start
- RAID 1 mode


![Naspi](./img/naspi.png)

## First Setup – Raspberry Pi OS + Manual Setup For Services

This setup was highly manual and required deeper system understanding.

### OS Choice

- Raspberry Pi OS Lite (64-bit, headless)
- OMV could not be installed directly, so RPi OS was required
- 64-bit necessary for certain containers/plugins

### Docker & Portainer
Containers were managed via Portainer (Docker backend, not Podman).
Portainer made orchestration easier, but required manual networking and volume configuration.

### Services
#### Homer (Dashboard)
- Used instead of Heimdall (PHP-based)
- Requires correct environment variables
- Central landing page for all services

#### Nextcloud
```bash
docker run \
--init \
--sig-proxy=false \
--name nextcloud-aio-mastercontainer \
--restart always \
--publish 80:80 \
--publish 8080:8080 \
--publish 8443:8443 \
--volume nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
--volume /var/run/docker.sock:/var/run/docker.sock:ro \
nextcloud/all-in-one:latest
```

#### Jellyfin

Container setup was straightforward.
The challenge was proper media folder mounting.

Instead of manually uploading via container volumes, I mounted shared folders directly via NFS.

#### Storage & Permissions (OMV)

Shared folders required additional ACL configuration.
Correct configuration path:
Storage → Shared Folders → Access Control List


Without proper ACL configuration:
- Either folders were accessible to everyone
- or inaccessible to everyone

Permissions via user panel did not work reliably - ACL solved it.

#### NFS Mounting

Mount entry in `/etc/fstab`:
```bash
192.168.178.82:/Media /home/homeserver/Media nfs defaults 0 0
```


NFS must be enabled in OMV and shared explicitly to the server IP.
Helpful guide:
https://linuxize.com/post/how-to-mount-an-nfs-share-in-linux/

#### SSH

SSH key export:
```bash
ssh-keygen -e -f ~/.ssh/id_rsa.pub
```
#### Pi-hole (Network DNS)

Port collisions can happen (Pi-hole uses common ports).

Configured local DNS entries:

192.168.178.20:8082 homeserver.net
192.168.178.20:9000 homeserver-portainer.net
192.168.178.20:1010 homeserver-pihole.net
192.168.178.20 homeserver-nas.net
192.168.178.20:8096 homeserver-jellyfin.net
192.168.178.20:9696 homeserver-prowlarr.net
192.168.178.20:8080 homeserver-qbittorrent.net
192.168.178.20:7878 homeserver-radarr.net
192.168.178.20:8989 homeserver-sonarr.net

**Important**:

Fritzbox configuration (IPv4 + IPv6) is critical.
If not configured correctly, the homeserver will not act as DNS.

#### Misc

Disable Raspberry Pi LEDs:
https://raspberrytips.com/disable-leds-on-raspberry-pi/

OMV setup reference:
https://www.youtube.com/watch?v=Y3yF1Rssudo

## Second (Current) Setup – Umbrel 

After running the manual setup for some time, I migrated everything to Umbrel.
**Why Umbrel?**
- Much cleaner UI
- App-store-like container management
- Less manual networking configuration
- Faster service deployment

The networking architecture (static IP, DNS via Pi-hole, Fritzbox configuration) remained mostly unchanged.

![umbrel](./img/umbrel.png)
source: © umbrel

### What I Learned

- Deep understanding of Linux permissions & ACL
- Docker networking & volume management
- NFS configuration and pitfalls
- DNS resolution & local domain mapping
- Router-level IPv4/IPv6 configuration
- RAID setup on Raspberry Pi hardware
- Tradeoffs between manual infrastructure vs platform abstraction

### Current Status

The server now runs stable under Umbrel with:
- Media streaming (Jellyfin)
- Cloud storage (Nextcloud)
- DNS blocking (Pi-hole)
- Download automation stack (Sonarr / Radarr / Prowlarr / qBittorrent)

This project was less about “just running services” and more about understanding the full infrastructure stack behind a self-hosted system.
