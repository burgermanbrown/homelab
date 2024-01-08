# Dell Precision Tower Homelab Server
Install guide/progress on a self-hosed virtualization server built on a Dell Precision Tower 7820

## Hardware/Software:

| Component | Specification |
|-----------|---------------|
|CPU|40 Core Intel Xeon Gold 6138 2.0GHz|
|RAM| 64GB (2 x 32GB) DDR4 RDIMM|
|STORAGE| 800GB SSD, RAID Z1 ZPool (3x Western Digital 2TB WD Blue PC Hard Drive - 5400 RPM Class, SATA 6 Gb/s, , 256 MB Cache, 3.5" - WD20EZAZ) |
|GPU| EVGA SUPERCLOCKED GTX 1080 8GB|
|OS|Debian Bookworm (Headless)|
|VIRTUALIZATION CLIENT|VMware Workstation 17 Pro|

## Goals
- [x] Successfuly boot into OS and configure Cockpit for access via web interface
- [ ] Configure VMWare
- [ ] Configure SSH server
- [ ] Setup Plex for video and music streaming
- [ ] Setup nextcloud instance for file hosting
- [ ] Configure webserver for hosting public facing website

## Progress Log - 12/29/2023 - It's Alive!

After helping the UPS driver get this out the truck and discarding the enormous packaging, the device is ready to be provisioned! Before installing the OS I needed to make some changes to the BIOS:
- Disable secure boot to mitigate headaches during setup (this may no longer be a problem post Bookworm, but it can't hurt)
- Extend BIOS Post time (Modern hardware is wicked fast, and sometimes this can make hitting key shortcuts for boot options and bios difficult).

With the USB Drive recognized, we can proceed with the OS installation. I will be running Debian headless since I'll be working from the command line or web interface anyway. The less clutter, the better. 

This host will be named "Venture" (venture.homelab.srv) in honor of the classic Adult Swim series. After partitioning the system drive and configuring the root and user accounts, we are presented with our first challenge:
```
brown@venture: sudo su
-bash: sudo: command not found
brown@venture:
```
It is clear that the main user on the machine is not in the super user group and thus cannot elevate their commands with ```sudo```. It is important that we remedy this instead of just switching to the root user. Working as root when installing packages, making changes to the system, etc. is dangerous when you have unchecked access to **ALL** resources on the system. In this instance we will need to switch to root and make the necessary changes for "brown" in the user groups:

By default sudo is not installed on Debian, but you can install it. First, we will need to sign the current user out with ```exit``` and then logging in as ```root``` with password we made during setup.

Then enable su-mode:
```
su -
```
Install sudo by running:
```
apt-get install sudo -y
```
After that give sudo rights to your own user:
```
usermod -aG sudo $username
```
With our user configured for sudo, we can now proceed with installing the Cockpit package. To do so we run the command ```sudo apt install cockpit```.

### Setting up Cockpit

Post installation, Cockpit’s service doesn’t automatically activate. You need to implement a few ```systemctl``` commands to initiate and enable Cockpit on your Debian system.

**Step 1: Activate the Cockpit service**
```
sudo systemctl start cockpit.socket
```
**Step 2: Enabling Cockpit on System Boot**

For automatic startup of Cockpit whenever your system boots, execute the following command:
```
sudo systemctl enable cockpit.socket
```
**Step 3: Verifying Cockpit Service Status**

In the final step, confirming if Cockpit is running as expected on your system is crucial. You can check the service status of Cockpit using the next command:
```
systemctl status cockpit.socket
```
When the setup is correctly done, you should see the service status indicated as active. This ensures that you have a fully functioning Cockpit web interface on your Debian system.

We will also need to configure the Uncomplicated Firewall (UFW) to permit Cockpit access through the firewall. Cockpit listens on port 9090 by default, and we must ensure that the firewall rules allow incoming connections on this port.

**Step 1: Assessing UFW Status**

The first task is to ascertain the current status of the UFW firewall. This is critical before making any changes to avoid compromising security. Run the following command to verify the status:
```
sudo ufw status
```
Activation is necessary to secure your Debian system if the firewall is inactive. For this, use the following command:
```
sudo ufw enable
```
In the event your Debian system does not have UFW installed, it can be installed by executing:
```
sudo apt install ufw
```
**Step 2: Granting Cockpit Access Through Firewall**

The next step is to configure the UFW firewall to permit incoming connections for Cockpit. Execute the following command to achieve this:
```
sudo ufw allow 9090
```
If successful, the command will yield the following output:
```
Rules updated
Rules updated (v6)
The displayed output signifies that the firewall rules for both IPv4 and IPv6 are updated, allowing traffic on port 9090.
```
**Step 3: Verifying the Firewall Configuration**

It’s vital to verify that the firewall configuration is correctly set up and grants access to Cockpit. To check the UFW rules, run the following command:
```
sudo ufw status
```
The output should indicate that port 9090 is open for incoming connections.

Once complete, we should be able to reach the web interface from your browser by entering the web address displayed at the login screen ```https://xxx.xxx.xx.xx:9090/```. 
