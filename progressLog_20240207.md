# Progress Log - 2024/04/07 - Aaaand We're Back
It's been a while, but unfortunately work has not been generous with leaving me time to work on this project. I've switched to Fedora as my daily OS, so getting my files hosted on this server is high priority. Unfortunately, Microsoft 365 integration on linux is abysmal, but I was shooting for self hosting anyway.

I've purchased four 12TB Seagate IronWolf drives for a whopping total 48TB for personal storage. I've also added several SSDs lying around to use for the VM instances I hope to create later. The Z-Pool we're creating today is dedicated solely for my file server (videos, music, backups, the works).

With our excuses and whatnot out of the way, lets jump into today's changes.

## Installing ZFS and Creating a Pool
We'll be relying on the very verbose and well documented [Debian Offical Wiki](https://wiki.debian.org/) for next few segments, but I will be documenting any challenges or changes that are needed to proceed with a smooth installation.

ZFS on Linux is provided in the form of DKMS source for Debian users. It is necessary to add the contrib section to your apt sources configuration to be able to get the packages. Also, it is recommended by Debian ZFS on Linux Team to install ZFS related packages from Backports archive. Upstream stable patches will be tracked and compatibility is always maintained. When configured, use following commands to install the packages:
```
sudo apt update;
sudo apt install linux-headers-amd64; 
sudo apt install -t stable-backports zfsutils-linux
```
We are then presented with our first issue in the subsequent output:
```
Hit:1 http://security.debian.org/debian-security bookworm-security InRelease
Hit:2 http://deb.debian.org/debian bookworm InRelease
Hit:3 http://deb.debian.org/debian bookworm-updates InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
linux-headers-amd64 is already the newest version (6.1.69-1).
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Reading package lists... Done
E: The value 'stable-backports' is invalid for APT::Default-Release as such a release is not available in the sources
```
Recieving this notice means we need to make sure /etc/apt/sources.list contains correct Debian codename backports:
```
codename=$(lsb_release -cs);echo "deb http://deb.debian.org/debian $codename-backports main contrib non-free"|sudo tee -a /etc/apt/sources.list && sudo apt update
```
We then should get the following output:
```
deb http://deb.debian.org/debian bookworm-backports main contrib non-free
Hit:1 http://security.debian.org/debian-security bookworm-security InRelease
Hit:2 http://deb.debian.org/debian bookworm InRelease
Hit:3 http://deb.debian.org/debian bookworm-updates InRelease
Get:4 http://deb.debian.org/debian bookworm-backports InRelease [56.5 kB]
Get:5 http://deb.debian.org/debian bookworm-backports/main amd64 Packages [172 kB]
Get:6 http://deb.debian.org/debian bookworm-backports/main Translation-en [139 kB]
Get:7 http://deb.debian.org/debian bookworm-backports/contrib amd64 Packages [5,368 B]
Get:8 http://deb.debian.org/debian bookworm-backports/contrib Translation-en [5,084 B]
Get:9 http://deb.debian.org/debian bookworm-backports/non-free amd64 Packages [1,284 B]
Get:10 http://deb.debian.org/debian bookworm-backports/non-free Translation-en [544 B]
Fetched 380 kB in 1s (279 kB/s)                                
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
```
We will then attempt the installation again:
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  build-essential cpp dkms dpkg-dev fakeroot g++ g++-12 gcc libalgorithm-diff-perl libalgorithm-diff-xs-perl libalgorithm-merge-perl libdpkg-perl libfakeroot libfile-fcntllock-perl
  libnvpair3linux libstdc++-12-dev libuutil3linux libzfs4linux libzpool5linux make patch python3-distutils python3-lib2to3 zfs-dkms zfs-zed
Suggested packages:
  cpp-doc menu debian-keyring g++-multilib g++-12-multilib gcc-12-doc gcc-multilib autoconf automake libtool flex bison gdb gcc-doc git bzr libstdc++-12-doc make-doc ed diffutils-doc
  debhelper nfs-kernel-server samba-common-bin zfs-initramfs | zfs-dracut
The following NEW packages will be installed:
  build-essential cpp dkms dpkg-dev fakeroot g++ g++-12 gcc libalgorithm-diff-perl libalgorithm-diff-xs-perl libalgorithm-merge-perl libdpkg-perl libfakeroot libfile-fcntllock-perl
  libnvpair3linux libstdc++-12-dev libuutil3linux libzfs4linux libzpool5linux make patch python3-distutils python3-lib2to3 zfs-dkms zfs-zed zfsutils-linux
0 upgraded, 26 newly installed, 0 to remove and 24 not upgraded.
Need to get 20.4 MB of archives.
After this operation, 91.0 MB of additional disk space will be used.
Do you want to continue? [Y/n]
```
Confirm with 'Y' and proceed with the installation, you'll eventually be presented with a scary message as shown below:

![image](https://github.com/burgermanbrown/homelab/assets/104027884/7d485d60-89e3-41bd-8040-2bd13df40ba3)

Don't fret, this doesn't apply to us since we wont be distributing OpenZFS and DKMS in any images. Press 'enter' and proceed.

**NOTE: This can take some time, be patient even if it hangs for a little while. If you're like me, you won't have some modules built on your OS by default, so the installer will need to create them as well.**

Once finished, we're ready to proceed to the creation of the pool.

Or so I thought...

I am working out of the Cockpit terminal, so I was informed that one of the ZFS services ```zfs-load-module.service``` had failed to start.

This is remediable with the ```systemctl``` command as shown below:

First we check the status of the service
```
systemctl status zfs-load-module.service
```
Our output should yield the following:
```
× zfs-load-module.service - Install ZFS kernel module
     Loaded: loaded (/lib/systemd/system/zfs-load-module.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Wed 2024-02-07 20:33:28 EST; 14min ago
   Main PID: 15874 (code=exited, status=1/FAILURE)
        CPU: 7ms
```
We can see that the service has failed to start, so we'll try restarting it.
```
systemctl restart zfs-load-module.service
```
Our output will now show that the service is now running:
```
● zfs-load-module.service - Install ZFS kernel module
     Loaded: loaded (/lib/systemd/system/zfs-load-module.service; enabled; preset: enabled)
     Active: active (exited) since Wed 2024-02-07 20:48:48 EST; 4s ago
    Process: 43430 ExecStart=/sbin/modprobe zfs (code=exited, status=0/SUCCESS)
   Main PID: 43430 (code=exited, status=0/SUCCESS)
        CPU: 1.538s
```

## Creating the Pool
Many disks can be added to a storage pool, and ZFS can allocate space from it, so the first step of using ZFS is creating a pool. It is recommended to use more than 1 whole disk to take advantage of full benefits, but it's fine to proceed with only one device or just a partition.

In the world of ZFS, device names with path/id are typically used to identify a disk, because the device names like /dev/sdX may change on every start. These names can be retrieved with ls -l /dev/disk/by-id/ or ls -l /dev/disk/by-path/

In case of using whole disks ZFS will automatically reserve 8 MiB at the end of the device, to allow for replacement and/or additional physical devices that don't have the exact same size as the other devices in the pool.

### Basic Configuration

Many disks can be added to a storage pool, and ZFS can allocate space from it, so the first step of using ZFS is creating a pool. It is recommended to use more than 1 whole disk to take advantage of full benefits, but it's fine to proceed with only one device or just a partition.

In the world of ZFS, device names with path/id are typically used to identify a disk, because the device names like /dev/sdX may change on every start. These names can be retrieved with
```ls -l /dev/disk/by-id/``` or ```ls -l /dev/disk/by-path/```

In case of using whole disks ZFS will automatically reserve 8 MiB at the end of the device, to allow for replacement and/or additional physical devices that don't have the exact same size as the other devices in the pool.

When using partitions (or preparing the disk manually in advance) it is possible to also use the GPT partition labels to identify the partition/disk, as they are customizable and nicer for humans to understand. These can be found in ```/dev/disk/by-partlabel/```.

We will be creating a mirrored pool with all four disks, with 2 disks per mirror:

We will start by getting the list of disks on our device.
```
thaddeus@gargantua:~$ ls -l /dev/disk/by-id/
```

Our output yields the following:
```
total 0
lrwxrwxrwx 1 root root  9 Feb  1 00:20 ata-CT480BX500SSD1_1929E18F7AA1 -> ../../sdg
lrwxrwxrwx 1 root root  9 Feb  1 00:20 ata-INTEL_SSDSC2BA800G3E_BTTV4133018G800JGN -> ../../sdf
lrwxrwxrwx 1 root root 10 Feb  1 00:20 ata-INTEL_SSDSC2BA800G3E_BTTV4133018G800JGN-part1 -> ../../sdf1
lrwxrwxrwx 1 root root 10 Feb  1 00:20 ata-INTEL_SSDSC2BA800G3E_BTTV4133018G800JGN-part2 -> ../../sdf2
lrwxrwxrwx 1 root root 10 Feb  1 00:20 ata-INTEL_SSDSC2BA800G3E_BTTV4133018G800JGN-part3 -> ../../sdf3
lrwxrwxrwx 1 root root  9 Feb  1 00:20 ata-SPCCSolidStateDisk_AA000000000000000592 -> ../../sdd
lrwxrwxrwx 1 root root  9 Feb  1 00:20 ata-ST12000VN0008-2YS101_ZRT0XFVM -> ../../sdc
lrwxrwxrwx 1 root root  9 Feb  1 00:20 ata-ST12000VN0008-2YS101_ZRT122M3 -> ../../sda
lrwxrwxrwx 1 root root  9 Feb  1 00:20 ata-ST12000VN0008-2YS101_ZV70GYDY -> ../../sdb
lrwxrwxrwx 1 root root  9 Feb  1 00:20 ata-ST12000VN0008-2YS101_ZV70GYWW -> ../../sde
lrwxrwxrwx 1 root root  9 Feb  1 00:20 usb-Generic-_SD_MMC_CRW_28203008282014000-0:0 -> ../../sdh
lrwxrwxrwx 1 root root  9 Feb  1 00:20 wwn-0x5000c500e7efa344 -> ../../sde
lrwxrwxrwx 1 root root  9 Feb  1 00:20 wwn-0x5000c500e7f5cb96 -> ../../sda
lrwxrwxrwx 1 root root  9 Feb  1 00:20 wwn-0x5000c500e806e3b8 -> ../../sdc
lrwxrwxrwx 1 root root  9 Feb  1 00:20 wwn-0x5000c500e8096707 -> ../../sdb
lrwxrwxrwx 1 root root  9 Feb  1 00:20 wwn-0x55cd2e404b5f9de1 -> ../../sdf
lrwxrwxrwx 1 root root 10 Feb  1 00:20 wwn-0x55cd2e404b5f9de1-part1 -> ../../sdf1
lrwxrwxrwx 1 root root 10 Feb  1 00:20 wwn-0x55cd2e404b5f9de1-part2 -> ../../sdf2
lrwxrwxrwx 1 root root 10 Feb  1 00:20 wwn-0x55cd2e404b5f9de1-part3 -> ../../sdf3
```

We know sda, sdb, sdc and sde to be the drives we wish to configure for mirrors, so grabbing their respective IDs we can create two pools called "bucket" aand "bucket-backup" with the four drives:

**bucket**:
```
sudo zpool create bucket mirror ata-ST12000VN0008-2YS101_ZRT122M3 ata-ST12000VN0008-2YS101_ZV70GYDY
```
**bucket-backup**:
```
zpool add bucket ata-ST12000VN0008-2YS101_ZRT0XFVM ata-ST12000VN0008-2YS101_ZV70GYWW
```
Entering ```zpool list``` will output our newly created mirros:

