# Dell Precision Tower Homelab Server
Install guide/progress on a self-hosed virtualization server built on a Dell Precision Tower 7820

## Hardware/Software:
Dell Precision Tower T7820
| Component | Specification |
|-----------|---------------|
|CPU|40 Core Intel Xeon Gold 6138 2.0GHz|
|RAM| 64GB (2 x 32GB) DDR4 RDIMM|
|STORAGE| 1.8TB SSD, RAID Z1 ZPool (4x Seagate IronWolf 12TB NAS Internal Hard Drive HDD – 3.5 Inch SATA 6Gb/s 7200 RPM 256MB Cache |
|GPU| EVGA SUPERCLOCKED GTX 1080 8GB|
|OS|Debian 12 Bookworm (Headless)|
|VIRTUALIZATION CLIENT|VMware Workstation 17 Pro|
| APPLICATIONS | Plex, Nextcloud, etc. |

Rasberry Pi 4

## Goals
- [x] Successfuly boot into OS and configure Cockpit for access via web interface
- [ ] Configure Nextcloud for local and remote access
- [ ] Configure Plex server for local and remote access
- [ ] Configure PiHole on Rasberry Pi
