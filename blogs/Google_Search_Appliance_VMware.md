# Google Search Appliance (GSA 7.6.512) on VMware - Complete Installation Guide

![GSA Logo](https://img.shields.io/badge/GSA-7.6.512-blue) ![VMware](https://img.shields.io/badge/VMware-Compatible-green) ![Arch Linux](https://img.shields.io/badge/Arch-Linux-blue)

## ⚠️ Important Security Notice

**This guide covers the installation of an abandoned, legacy product with known security vulnerabilities.** 

- **Never expose the GSA to the public internet**
- Keep it isolated on a dedicated VLAN, host-only network, or VPN
- Use only for research, archival, or educational purposes
- Update your security documentation and risk assessments accordingly

## 📋 Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Architecture Overview](#architecture-overview)
- [Step 1: VMware VM Setup](#step-1-vmware-vm-setup)
- [Step 2: Arch Linux Installation](#step-2-arch-linux-installation)
- [Step 3: Post-Installation Setup](#step-3-post-installation-setup)
- [Step 4: Environment Verification](#step-4-environment-verification)
- [Step 5: GSA Build Process](#step-5-gsa-build-process)
- [Step 6: Boot GSA Appliance](#step-6-boot-gsa-appliance)
- [Step 7: Network Configuration](#step-7-network-configuration)
- [Step 8: Admin Access & Login](#step-8-admin-access--login)
- [Step 9: Testing with Feeds](#step-9-testing-with-feeds)
- [Troubleshooting](#troubleshooting)
- [Security Hardening](#security-hardening)

## Overview

This guide provides a complete walkthrough for installing Google Search Appliance (GSA) 7.6.512 on VMware using Arch Linux as a build environment. The process involves:

1. Creating a dual-disk VMware VM
2. Installing Arch Linux as a build environment
3. Using GSABuilder to create the appliance
4. Configuring networking and accessing the admin interface

## Prerequisites

### Software Requirements
- **VMware Workstation Pro/Player** (or VMware ESXi/vSphere)
- **Arch Linux ISO** (latest version)
- **20+ GB free disk space** for downloads and temporary files
- **Stable internet connection** (several GB download required)

### Hardware Requirements
- **CPU**: 2+ cores recommended
- **RAM**: 16 GB recommended (8 GB minimum)
- **Storage**: 200+ GB total across both disks
- **Network**: 2 virtual NICs required

### Knowledge Prerequisites
- Basic Linux command line experience
- VMware virtual machine management
- Understanding of network concepts (VLANs, subnets)

## Architecture Overview

The installation uses a dual-disk setup:

```
┌─────────────────┐    ┌─────────────────┐
│   Disk 1        │    │   Disk 2        │
│   (TARGET)      │    │  (WORKSPACE)    │
│                 │    │                 │
│  → GSA          │    │  Arch Linux     │
│    Appliance    │    │  Build Env      │
│  120-200 GB     │    │  40-60 GB       │
└─────────────────┘    └─────────────────┘

┌─────────────────┐    ┌─────────────────┐
│   eth0 (NIC1)   │    │   eth1 (NIC2)   │
│   Bridge/NAT    │    │   Host-Only     │
│                 │    │                 │
│  Admin Access   │    │  Setup Wizard   │
│  192.168.x.x    │    │  192.168.255.1  │
└─────────────────┘    └─────────────────┘
```

## Step 1: VMware VM Setup

### 1.1 Create Virtual Machine

1. **Open VMware Workstation/Player**
2. **Create New Virtual Machine** → Custom (advanced)
3. **Hardware Compatibility**: Latest available
4. **Guest OS**: Linux → Other Linux 5.x kernel 64-bit
5. **VM Name**: `GSA-Builder` (or your preference)
6. **Firmware**: BIOS/Legacy (**Important**: Disable Secure Boot)
7. **Processors**: 2 cores minimum (4+ recommended)
8. **Memory**: 16 GB recommended (8 GB minimum)

### 1.2 Configure Storage

Add two virtual disks:

**Disk 1 (TARGET - Will become GSA)**:
- **Size**: 120-200 GB
- **Location**: SCSI 0:0
- **Type**: Thick provisioned (recommended) or Thin
- **Split files**: No (single file preferred)

**Disk 2 (WORKSPACE - Arch Linux)**:
- **Size**: 40-60 GB  
- **Location**: SCSI 0:1
- **Type**: Thick or Thin provisioned
- **Split files**: No

### 1.3 Network Configuration

**Add Network Adapter 1 (eth0)**:
- **Connection**: Bridged or NAT (your choice)
- **Adapter Type**: E1000E (recommended for compatibility)

**Add Network Adapter 2 (eth1)**:
- **Connection**: Host-only (VMnet1)
- **Adapter Type**: E1000E

### 1.4 Configure Host-Only Network

1. **VMware** → **Edit** → **Virtual Network Editor** (Run as Administrator)
2. **Select VMnet1** (Host-only)
3. **Configure**:
   - **Subnet IP**: `192.168.255.0`
   - **Subnet mask**: `255.255.255.0`
   - **Uncheck**: "Use local DHCP service"
4. **Apply**

**Configure Windows Host Adapter**:
1. **Network Settings** → **Change adapter settings**
2. **Right-click "VMware Network Adapter VMnet1"** → Properties
3. **IPv4 Settings**:
   - **IP**: `192.168.255.2`
   - **Mask**: `255.255.255.0`
   - **Gateway**: (leave blank)
   - **DNS**: (leave blank)

### 1.5 Attach Arch ISO

1. **VM Settings** → **CD/DVD**
2. **Use ISO image file** → Browse to Arch Linux ISO
3. **Connect at power on**: Checked

## Step 2: Arch Linux Installation

### 2.1 Boot from ISO

1. **Power on VM**
2. **At Arch boot menu**: Press `e` to edit
3. **Append to kernel line**: `net.ifnames=0 biosdevname=0`
4. **Boot** (this ensures eth0/eth1 naming)

### 2.2 Run Arch Installer

At the Arch command prompt:

```bash
archinstall
```

### 2.3 Installation Configuration

**Important Settings** (use arrow keys and Enter to navigate):

| Setting | Value | Notes |
|---------|--------|--------|
| **Drives** | Select 40-60 GB disk (usually `/dev/sdb`) | Choose "Erase all", ext4, no swap |
| **Bootloader** | GRUB | Default is fine |
| **Networking** | NetworkManager | Required for internet access |
| **User Account** | Create admin user | Mark as superuser, set root password |
| **Profile** | Minimal | Reduces installation time |

**Extra Packages** (copy/paste this entire list):
```
base linux linux-firmware vim git curl pv gnupg tar util-linux parted lvm2 e2fsprogs rpm-tools protobuf python python-pip python-protobuf jq wget
```

### 2.4 Complete Installation

1. **Review settings** and confirm
2. **Install** (15-30 minutes depending on internet)
3. **Reboot** when complete
4. **Remove ISO** from VM settings or ensure boot priority is set to hard disk

## Step 3: Post-Installation Setup

### 3.1 First Boot Setup

Log in with your created user account.

**Make NIC naming permanent**:
```bash
sudo sed -i 's/^\(GRUB_CMDLINE_LINUX_DEFAULT="\)/\1net.ifnames=0 biosdevname=0 /' /etc/default/grub
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

**Enable networking and time sync**:
```bash
sudo systemctl enable --now NetworkManager
sudo timedatectl set-ntp true
```

### 3.2 Create Workspace Directories

The build process requires significant temporary space:

```bash
sudo mkdir -p /srv/work/{tmp,gnupg,home}
sudo chmod 1777 /srv/work/tmp
sudo chmod 700  /srv/work/gnupg
```

### 3.3 Optional: Create Swapfile

If RAM is limited, create a swapfile:

```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
sudo swapon -a
```

### 3.4 Reboot to Apply Changes

```bash
sudo reboot
```

## Step 4: Environment Verification

After reboot, verify the environment:

**Check network interfaces**:
```bash
ip -br link
# Should show: lo, eth0, eth1
```

**Check disk space**:
```bash
df -h /srv/work
# Should show 30+ GB available
```

**Update system and install prerequisites**:
```bash
sudo pacman -Syu --noconfirm
sudo pacman -S --noconfirm \
  python python-pip git curl pv gnupg tar util-linux parted lvm2 e2fsprogs \
  rpm-tools protobuf python-protobuf jq wget
```

## Step 5: GSA Build Process

### 5.1 Clone GSABuilder Repository

```bash
cd /srv/work
git clone https://github.com/ChlorideCull/GSABuilder.git
cd GSABuilder
```

### 5.2 Setup Python Environment

```bash
# Create virtual environment (optional but recommended)
python -m venv .venv && source .venv/bin/activate

# Configure environment variables for large file operations
export HOME=/srv/work/home
export HISTFILE=$HOME/.bash_history
export TMPDIR=/srv/work/tmp
export GNUPGHOME=/srv/work/gnupg
```

### 5.3 Identify Target Disk

**⚠️ CRITICAL: Verify which disk is which before proceeding!**

```bash
lsblk -o NAME,SIZE,TYPE,MOUNTPOINTS
```

Example output:
```
NAME   SIZE   TYPE MOUNTPOINTS
sda    120G   disk              ← TARGET (empty, will be wiped)
sdb     40G   disk
├─sdb1  1G    part /boot
└─sdb2  39G   part /            ← Arch OS (mounted)
```

- **TARGET disk**: No mountpoints (usually `/dev/sda`)
- **OS disk**: Shows `/` mounted (usually `/dev/sdb`)

### 5.4 Run the Build

**Replace `sda` with your actual target disk identifier:**

```bash
sudo -E "$(pwd)/.venv/bin/python" build.py --generalize sda
```

**Build Process Notes**:
- Downloads several GB of Google software packages
- Process can take 1-3 hours depending on internet speed
- GPG warnings about legacy keys are normal
- "blkdiscard not supported" warnings are harmless
- Monitor `/srv/work/tmp` to ensure space doesn't fill up

### 5.5 Build Completion

When successful, you'll see:
```
[INFO] Build completed successfully
[INFO] GSA appliance ready on /dev/sda
```

**Save the credentials**:
```bash
cat /srv/work/GSABuilder/credentials.json
# Copy this output - contains admin password
```

**Reboot**:
```bash
sudo reboot
```

## Step 6: Boot GSA Appliance

### 6.1 Modify Boot Order

1. **Power off VM**
2. **VM Settings** → **Hard Disk**
3. **Advanced** → **SCSI Node**: Make GSA disk `0:0` (first)
4. **Or temporarily remove Arch disk** (just detach, don't delete VMDK)

### 6.2 First Boot

1. **Power on VM**
2. **Wait 5-20 minutes** for initial appliance setup
3. **Console should show GSA boot messages**

The appliance will configure itself and start services automatically.

## Step 7: Network Configuration

### 7.1 Verify Network Interfaces

On the GSA console, check IP assignments:
```bash
# From GSA console (if accessible)
ifconfig
```

Expected configuration:
- **eth0**: DHCP IP from your LAN (e.g., `192.168.146.x`)
- **eth1**: Static IP `192.168.255.1`

### 7.2 Network Access Points

The GSA provides multiple access methods:

| Interface | URL | Purpose |
|-----------|-----|---------|
| **Admin Console (HTTPS)** | `https://<eth0-ip>:8443` | Main admin interface (preferred) |
| **Admin Console (HTTP)** | `http://<eth0-ip>:8000` | Alternative admin interface |
| **First-Boot Wizard** | `http://192.168.255.1:1111` | Initial setup wizard |

## Step 8: Admin Access & Login

### 8.1 Access Admin Console

1. **Find eth0 IP**: Check your router/DHCP server or VM network info
2. **Browse to**: `https://<eth0-ip>:8443`
3. **Accept certificate warning** (self-signed certificate)

### 8.2 Login Credentials

- **Username**: `admin`
- **Password**: Found in `/srv/work/GSABuilder/credentials.json`

If you no longer have the Arch disk accessible:
1. **Reattach Arch disk** as second drive
2. **Boot Arch** temporarily to retrieve credentials
3. **Or mount Arch disk** from another Linux system

### 8.3 Initial Configuration

**Immediately after first login**:
1. **Change admin password**
2. **Review system status**
3. **Configure basic settings**

## Step 9: Testing with Feeds

### 9.1 Create Data Source

1. **Admin Console** → **Content Sources** → **Feeds**
2. **Add data source**:
   - **Name**: `test-feed`
   - **Trusted feed IPs**: Add your feed source IP
   - **Temporarily disable** "Require secure feeds" for testing

### 9.2 Test Feed Document

Create `test-feed.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<gsafeed>
  <header>
    <datasource>test-feed</datasource>
    <feedtype>incremental</feedtype>
  </header>
  <group>
    <record url="test://example/doc-1"
            mimetype="text/html"
            action="add"
            authmethod="none"
            last-modified="2025-01-01T12:00:00Z">
      <metadata>
        <meta name="tags" content="test;demo"/>
        <meta name="category" content="documentation"/>
      </metadata>
      <content encoding="base64binary">
        PCFET0NUWVBFIGh0bWw+PGh0bWw+PGhlYWQ+PHRpdGxlPlRlc3QgRG9jdW1lbnQ8L3RpdGxlPjwvaGVhZD48Ym9keT48aDE+VGVzdCBDb250ZW50PC9oMT48cD5UaGlzIGlzIGEgdGVzdCBkb2N1bWVudCBmb3IgR1NBIGY=
      </content>
    </record>
  </group>
</gsafeed>
```

### 9.3 Submit Feed

```bash
curl -H "Content-Type: text/xml" \
     --data-binary @test-feed.xml \
     http://<eth0-ip>:19900/xmlfeed
```

### 9.4 Verify Ingestion

1. **Admin Console** → **Content Sources** → **Feeds** → **Monitor**
2. **Check for**: "Documents added: 1"
3. **Search interface**: Try searching for "test content"

## Troubleshooting

### Common Issues and Solutions

#### Build Issues

**"No space left on device" during build**:
```bash
# Check workspace usage
df -h /srv/work
du -sh /srv/work/tmp/*

# Clean temporary files if needed
sudo rm -rf /srv/work/tmp/*
```

**Wrong disk targeted**:
```bash
# Always verify before building
lsblk -o NAME,SIZE,TYPE,MOUNTPOINTS
# TARGET disk should show NO mountpoints
```

#### Network Issues

**eth0/eth1 not present**:
```bash
# Check interface names
ip link show

# Rename interfaces if needed
sudo ip link set <current-name> down
sudo ip link set <current-name> name eth0
sudo ip link set eth0 up
```

**Port 1111 not responding**:
- System may already be configured
- Use admin console on :8443 or :8000 instead
- Check VMware host-only network configuration

#### Access Issues

**Admin password unknown**:
1. **Reattach Arch disk**
2. **Boot into Arch**
3. **Read credentials**:
   ```bash
   cat /srv/work/GSABuilder/credentials.json
   ```

**Cannot access admin console**:
```bash
# From GSA console, check services
ps aux | grep -i gsa
netstat -tlnp | grep :8443
```

### Debug Commands

**Check GSA status**:
```bash
# On GSA appliance console
service --status-all
tail -f /var/log/messages
```

**Network connectivity**:
```bash
# From host or another machine
ping <gsa-ip>
telnet <gsa-ip> 8443
nmap -p 8000,8443,19900 <gsa-ip>
```

## Security Hardening

### Essential Security Steps

1. **Network Isolation**:
   ```bash
   # Configure firewall rules
   iptables -A INPUT -s <trusted-subnet>/24 -j ACCEPT
   iptables -A INPUT -j DROP
   ```

2. **Change Default Passwords**:
   - Admin console password
   - SSH console password (if enabled)
   - Root password

3. **Disable Unnecessary Services**:
   ```bash
   # Review running services
   service --status-all
   
   # Disable unneeded services
   service <service-name> stop
   chkconfig <service-name> off
   ```

4. **Enable Secure Feeds**:
   - Re-enable "Require secure feeds"
   - Configure IP restrictions
   - Use HTTPS where possible

### Monitoring and Logging

**Setup log monitoring**:
```bash
# Monitor key log files
tail -f /var/log/messages
tail -f /var/log/secure
```

**Regular security checks**:
- Review user access logs
- Monitor feed ingestion
- Check system resource usage

## Additional Resources

### Useful Links
- [GSABuilder Repository](https://github.com/ChlorideCull/GSABuilder) - Send a Star for the Awesome Build Script
- [GSA Documentation Archive](https://archive.org/details/google-search-appliance-documentation)
- [VMware Workstation Documentation](https://docs.vmware.com/en/VMware-Workstation-Pro/)

### Community Support
- Search for "GSA" and "Google Search Appliance" in relevant forums
- Archive.org has historical documentation
- Consider reaching out to enterprise search communities

---

## 📝 Contributing

Found an issue or improvement? Please:
1. Open an issue describing the problem
2. Submit a pull request with fixes
3. Share your experience in discussions

## ⚖️ Legal Notice

This guide is for educational and archival purposes. The Google Search Appliance is discontinued software. Users are responsible for:
- Compliance with software licensing
- Network security and isolation
- Appropriate use cases
- Regular security updates and monitoring

---
