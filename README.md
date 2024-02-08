# Dell Precision Tower Homelab Server
Install guide/progress on a self-hosed virtualization server built on a Dell Precision Tower 7820

## Hardware/Software:

| Component | Specification |
|-----------|---------------|
|CPU|40 Core Intel Xeon Gold 6138 2.0GHz|
|RAM| 64GB (2 x 32GB) DDR4 RDIMM|
|STORAGE| 1.8TB SSD, RAID Z1 ZPool (4x Seagate IronWolf 12TB NAS Internal Hard Drive HDD – 3.5 Inch SATA 6Gb/s 7200 RPM 256MB Cache |
|GPU| EVGA SUPERCLOCKED GTX 1080 8GB|
|OS|Debian Bookworm (Headless)|
|VIRTUALIZATION CLIENT|VMware Workstation 17 Pro|
| APPLICATIONS | Plex, Nextcloud, qBittorrent, etc. |

## Goals
- [x] Successfuly boot into OS and configure Cockpit for access via web interface
- [ ] Configure ZFS
- [ ] Configure SSH server
- [ ] Setup nextcloud instance for file hosting
- [ ] Configure webserver for hosting public facing website
