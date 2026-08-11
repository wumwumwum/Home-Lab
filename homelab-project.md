# Homelab Project Portfolio

A record of self-hosted infrastructure, virtualization, and cybersecurity projects built across a home lab environment (Proxmox nodes, Ubuntu Server, and assorted repurposed hardware). Each entry documents the goal, outcome, and exact steps taken so the project can be reproduced.

## Table of Contents

**Core Infrastructure Projects**
- [1. Ubuntu Server 26.04 LTS — Lenovo Laptop](#1-ubuntu-server-2604-lts--lenovo-laptop)
- [2. Pi-hole on Repurposed ThinkPad *(Abandoned)*](#2-pi-hole-on-repurposed-thinkpad-abandoned)
- [3. Proxmox VE 9.2 — ThinkCentre 920s](#3-proxmox-ve-92--thinkcentre-920s-lgserver)
- [4. Windows Server 2022 VM in Proxmox](#4-windows-server-2022-vm-in-proxmox)
- [5. Ubuntu Server 26.04 LTS — Lenovo SFF PC](#5-ubuntu-server-2604-lts--lenovo-small-form-factor-pc)
- [6. Home Assistant in Proxmox](#6-home-assistant-in-proxmox)
- [7. Tailscale via Debian 12 LXC](#7-tailscale-via-debian-12-lxc)
- [8. Pi-hole in Proxmox (Ubuntu 24 LTS) + Network-wide DNS](#8-pi-hole-in-proxmox-ubuntu-24-lts--network-wide-dns)
- [9. Docker on Standalone Ubuntu Server](#9-docker-on-standalone-ubuntu-server)
- [10. Docker on Ubuntu Server Inside Proxmox](#10-docker-on-ubuntu-server-inside-proxmox)
- [11. VLAN + Linux Bridge Across Proxmox Nodes](#11-vlan--linux-bridge-across-proxmox-nodes)
- [12. TrueNAS in Proxmox](#12-truenas-in-proxmox)
- [13. Immich (Self-Hosted Photo Backup)](#13-immich-self-hosted-photo-backup)
- [14. Pi-hole over Debian (proxmoxBeta)](#14-pi-hole-over-debian-proxmoxbeta)
- [15. Metasploitable2 in Proxmox](#15-metasploitable2-in-proxmox)
- [16. Wazuh SIEM](#16-wazuh-siem)
- [17. Splunk SIEM](#17-splunk-siem)
- [18. ITSM-NG (Asset / Budget / Ticketing)](#18-itsm-ng-asset--budget--ticketing)
- [19. SnipeIT (Asset / Budget / Ticketing)](#19-snipeit-asset--budget--ticketing)

**Reference**
- [Quick Reference & Troubleshooting Notes](#quick-reference--troubleshooting-notes)
- [Links of Note](#links-of-note)
- [Skills Demonstrated](#skills-demonstrated)

---

## 1. Ubuntu Server 26.04 LTS — Lenovo Laptop

**Outcome:** ✅ Successful install

### Prerequisites
- Bootable USB drive with Ubuntu Server 26.04 LTS

### Installation
1. Boot from the Ubuntu Server 26.04 LTS USB.
2. Select **Try or Install Ubuntu Server** (initialization takes a couple of minutes).
3. Select language: **English**.
4. Select keyboard layout: accept the default.
5. Choose installation base: **Ubuntu Server**, plus **Search for third-party drivers**.
6. Configure at least one network interface and select **Done**.
   - *Note:* on a prior install, no connection was available, which severely limited the server. Make sure the router is configured before this step — this install initially hit that same issue and required reconfiguring the router mid-install.
   - Configure both wired and Wi-Fi connection types if available.
7. Proxy configuration: not applicable — select **Done**.
8. Ubuntu archive mirror configuration:
   - First attempt failed because the router wasn't properly configured. Fixed by resetting the router to factory defaults and reconfiguring SSIDs for 2.4 GHz, 5 GHz, and 5 GHz-2 bands.
   - Second attempt succeeded — select **Done**.
9. Guided storage configuration: **Use entire disk** → **Done**.
10. Storage configuration: **Done** → **Continue**.
11. Profile configuration: set hostname/username/password.
12. Upgrade to Ubuntu Pro: **Skip for now** → **Continue**.
13. SSH configuration: select **Install OpenSSH server** → **Done**.
14. Third-party drivers: **Continue**.
15. Featured server snaps — select and install the stable versions of:
    - `microk8s` (Kubernetes)
    - `nextcloud` (Nextcloud Server)
    - `wekan` (Kanban)
    - `powershell`
    - `wormhole`
    - `aws-cli`
    - `slcli`
    - `lxd`
    
    Select **Done**.
16. Installation runs.
17. On completion, select **View full log** to check for errors, **Close**, then **Reboot Now**.
18. System reboots (cursor blinks ~30 seconds).
19. At the prompt to remove the installation medium, remove the USB and press **Enter**.
20. System restarts.

### Post-Install Hardening
1. After restart, the system auto-updates the snaps selected during install (`lxd`, `microk8s`, `nextcloud`, `powershell`, `slcli`, `wekan`). Wait for this to finish before logging in.
2. Log in with the configured credentials once the SSH host key generation prompt completes.
3. **Update the system:**
   ```bash
   sudo apt update
   sudo apt upgrade
   ```
4. **Secure SSH access:**
   - Disable root login — edit `/etc/ssh/sshd_config`:
     ```bash
     sudo nano /etc/ssh/sshd_config
     ```
     Change `PermitRootLogin yes` to `PermitRootLogin no`.
   - Set up SSH key authentication from your local machine:
     ```bash
     ssh-copy-id username@your_server_ip
     ```
5. **Configure the firewall (UFW):**
   ```bash
   sudo ufw allow OpenSSH
   sudo ufw enable
   sudo ufw status
   ```
6. **Create a non-root user:**
   ```bash
   sudo adduser newusername
   sudo usermod -aG sudo newusername
   ```

### Optional: Add a Desktop GUI
1. Install the GUI:
   ```bash
   sudo apt install ubuntu-desktop
   ```
   Not required, and it does consume more resources, but it's a nice-to-have. Installation took roughly 1 hour 30 minutes.
2. Reboot and confirm the desktop environment loads and login works.
3. First-boot setup: skip Ubuntu Pro activation for now, disable Location Services, decline usage data sharing, choose **Dark** theme, and finish.

---

## 2. Pi-hole on Repurposed ThinkPad *(Abandoned)*

**Outcome:** ❌ Abandoned on this hardware — later installed as a Proxmox VM instead ([see Project #8](#8-pi-hole-in-proxmox-ubuntu-24-lts--network-wide-dns) / [#14](#14-pi-hole-over-debian-proxmoxbeta)).

1. Repurposed an old ThinkPad for the project.
2. Found a spare M.2 22x30mm SSD that happened to fit the laptop's slot.
3. Opened the case, installed the SSD, and reassembled.
4. Booted into the boot loader and ran a USB copy of Hiren's BootCD.
5. Ran file recovery tools (PhotoRec, DNDE, Puran) and TestDisk on the drive — nothing of value was recovered, so the drive was cleared for reuse.
6. Test-booted the drive: no boot — likely missing a bootloader. Decided to do a clean Ubuntu install instead.
7. Clean-installed Ubuntu 26.04 Desktop LTS:
   - Language: English
   - Accessibility: Next
   - Keyboard layout: English (US)
   - Connected to Wi-Fi
   - Selected interactive installation with default app selection
   - Selected **Install third-party software** and **Download and install support...**
   - Erased disk and installed Ubuntu
   - Enabled full-disk encryption with a passphrase
   - Created the local account and set timezone
   - Reviewed choices and started the install

**Project note:** The first install attempt failed due to a dead or damaged drive. Rather than keep troubleshooting the aging laptop hardware, the Pi-hole deployment was moved to a Proxmox VM.

---

## 3. Proxmox VE 9.2 — ThinkCentre 920s ("LgServer")

**Outcome:** ✅ Successful install

1. **Download the ISO** from the official Proxmox website (Proxmox VE 9.2).
2. **Create bootable media** — flash the ISO to USB with Balena Etcher and verify the flash succeeded.
3. **Boot the target machine** from the installation USB.
4. **Run the installation wizard:**
   - Select language.
   - Configure network (hostname, IP address).
   - Select storage target (chose to overwrite the drive).
   - Set the root password.
   - Review settings and confirm to begin installation.
5. **Access the web interface** once install completes, using the server's IP (found via `ip address` in the local terminal). Log in with the root account.
6. **Set up SSH:**
   ```bash
   ssh-keygen
   ```
   Accept the default key location, set a passphrase, and add the key to `authorized_keys`. Test access from another device:
   ```bash
   ssh username@IP
   ```
   (End a session with `Ctrl+D`.)
7. **Run the Proxmox VE Helper-Scripts post-install script** to disable enterprise repo prompts:
   ```bash
   bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/misc/post-pve-install.sh)"
   ```
8. **Install sudo and vim:**
   ```bash
   apt update
   apt install sudo
   apt install vim
   ```
9. **Create a non-root user** (stop using root for daily tasks):
   ```bash
   useradd -m -s /bin/bash USERNAME
   passwd USERNAME
   usermod -aG sudo USERNAME
   ```
10. **Grant Proxmox admin permissions to the new user:**
    ```bash
    pveum useradd dominic@pve -comment "New Admin User"
    pveum aclmod / -user dominic@pve -role Administrator
    ```
11. **Install Samba** and create a Samba user:
    ```bash
    sudo apt install samba
    sudo smbpasswd -a USERNAME
    ```

---

## 4. Windows Server 2022 VM in Proxmox

**Outcome:** ✅ Successful install

1. **Download the ISO via Proxmox:**
   - Datacenter → localhost → local (localhost) → ISO Images → **Download from URL**.
   - Paste the Windows Server 2022 ISO URL, **Query URL**, then **Download**.
   - Confirm success by reviewing the download log.
2. **Create the VM** with these specs:
   - CPU: 2 cores
   - RAM: 4 GB
   - Storage: 60 GB
   - BIOS: OVMF (UEFI)
   - Network: VirtIO
3. **Start the VM and install Windows Server 2022.**
   - If the installer doesn't see a disk, use **Load Driver** to mount the VirtIO driver ISO. Installation takes a while.
4. **Install the VirtIO SCSI driver** during setup so the disk is detected (driver source: [Fedora People VirtIO archive](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/)).
5. **Install the VirtIO network driver** after Windows installs, via Explorer → CD Drive (D:) → `virtio-win-gt-x64.exe`.
6. **Enable the QEMU Guest Agent:** in the Proxmox VM's **Options** tab, enable QEMU Guest Agent.
7. **Update Windows Server** and restart.
8. **Configure Windows Defender:** enable Tamper Protection and Controlled Folder Access.
9. **Add Roles and Features** as needed.
10. **Customize user permissions** in Local Group Policy.
11. **Set folder permissions.**

---

## 5. Ubuntu Server 26.04 LTS — Lenovo Small Form Factor PC

**Outcome:** ✅ Successful install

Same core flow as [Project #1](#1-ubuntu-server-2604-lts--lenovo-laptop), with these differences:
- Only a wired connection was configured (no Wi-Fi).
- The archive mirror step succeeded on the first attempt.
- No featured server snaps were selected during install.

Post-install hardening steps (system update, SSH lockdown, UFW, non-root user) were identical to Project #1 — see that section for exact commands.

---

## 6. Home Assistant in Proxmox

**Outcome:** ✅ Successful install

1. Install `pv`:
   ```bash
   sudo apt install pv
   ```
2. Install Home Assistant OS via the Proxmox shell (Datacenter → localhost → Shell):
   ```bash
   bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/vm/haos-vm.sh)"
   ```
3. Confirm the `haos` VM is available.
   - **Gotcha:** the console appeared blurred/unreadable — this turned out to be a Firefox rendering issue. Switching to Chromium as the default browser fixed it.
4. Access the Home Assistant web interface:
   - `http://<server-ip>:8123` — Home Assistant UI
   - `http://<server-ip>:4357` — Observer
5. Add an area.
6. Map devices.
7. Confirm devices respond via the mobile app.

---

## 7. Tailscale via Debian 12 LXC

**Outcome:** ✅ Successful install

1. **Download the Debian 12 CT template:** Datacenter → localhost → local (localhost) → CT Templates → Templates → find `debian-12-standard` → Download.
2. **Create the container:**
   - General: leave as an unassigned/unprivileged container, set a password.
   - Storage: select the Debian template.
   - Disk: 8 GB is sufficient.
   - CPU: 4 cores.
   - Memory: 8192 MB.
   - Network: defaults.
   - Leave **Start after created** unchecked on the final tab.
3. **Enable TUN/TAP device access for the LXC** (required for Tailscale):
   - Open the Proxmox shell and edit the container config:
     ```bash
     vi /etc/pve/lxc/{NODE_ID}.conf
     ```
   - Add:
     ```
     lxc.cgroup2.devices.allow: c 10:200 rwm
     lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
     ```
   - Save and quit in vi (`Esc`, `:wq`).
   - Start the container:
     ```bash
     pct start {NODE_ID}
     ```
4. **Run the container**, open its console, and log in as root.
5. **Update and install curl:**
   ```bash
   apt update
   apt upgrade
   apt install curl
   ```
6. **Install Tailscale:**
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   ```
7. **Bring Tailscale up:**
   ```bash
   tailscale up --ssh
   ```
   Open the printed URL in a browser to authenticate the device, then confirm it appears at [login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines).

---

## 8. Pi-hole in Proxmox (Ubuntu 24 LTS) + Network-wide DNS

**Outcome:** ✅ Successful install

> ⚠️ The command in step 5c below includes a live-looking Tailscale auth key from the original log. **Replace it with a placeholder (or your own key) before publishing** — do not commit real auth keys to a public repo.

### Ubuntu VM setup
1. In Proxmox, download the `ubuntu-standard-26.04` CT template (Datacenter → localhost → local → CT Templates → Templates).
2. Create the VM: name it, select the Ubuntu ISO, and leave System/Disks/CPU/Memory/Network at defaults. Review the confirmation screen for errors.
3. Run the Ubuntu Server installer: language English, default keyboard, base install **Ubuntu (minimal/stripped down)**, wired network only, default mirror, use entire disk, install OpenSSH server, skip Ubuntu Pro, skip third-party drivers prompt, and select **no** featured snaps.
4. Complete install, reboot, remove install media when prompted.
5. Log in and update:
   ```bash
   sudo apt update
   sudo apt upgrade
   ```
6. Set up SSH key auth:
   ```bash
   ssh-keygen
   ```
   Set a passphrase, then confirm you can SSH into the VM from your local terminal.

### Pi-hole install
1. SSH into the VM.
2. Install Pi-hole:
   ```bash
   curl -sSL https://install.pi-hole.net | bash
   ```
3. Follow the installer prompts:
   - Agree to the disclaimers.
   - **Set a static IP for this VM** — reserve it in the router's DHCP settings before continuing.
   - Choose an upstream DNS provider (Google was used here).
   - Enable blocklists and query logging.
   - Choose the privacy level (show everything was selected).
4. Note the admin URL shown at completion, e.g. `http://<pihole-ip>/admin`.
5. **Add the Pi-hole admin page to Tailscale:**
   - Tailscale admin → Machines → Add Device → Linux Server → generate the install script.
   - Run the generated script on the Pi-hole host, e.g.:
     ```bash
     curl -fsSL https://tailscale.com/install.sh | sh && sudo tailscale up --auth-key=<YOUR_AUTH_KEY>
     ```
   - Verify status:
     ```bash
     tailscale status
     ```
   - If prompted, authenticate via `sudo tailscale up` and the printed URL.
6. Set the Pi-hole admin password:
   ```bash
   sudo pihole setpassword
   ```

### Tailscale DNS configuration
1. In the Tailscale admin console, copy the Pi-hole machine's Tailscale IP.
2. Go to **DNS** → **Add nameserver** → **Custom**, paste the IP, and save.
3. Enable **Override DNS servers**.

### Network-wide device configuration
1. **Point the router's DNS at Pi-hole:** router admin → Advanced → Setup → Internet Setup → replace the DNS entries with the Pi-hole's IP as primary DNS.
2. **Enable DHCP on Pi-hole first**, *then* disable it on the router — the order matters, or DHCP handoff breaks. (This log includes a note that getting the order wrong required a factory reset of the router to recover.)
   - Pi-hole: Settings → DNS → Expert Mode → **Permit all origins**.
   - Pi-hole: Settings → DHCP → enable DHCP server, set the IP range (e.g. `192.168.1.2`–`192.168.1.254`), gateway, netmask (`255.255.255.0`), and a 24h lease time.
3. Disable the router's own DHCP server: Advanced → Setup → LAN Setup → uncheck "use router as DHCP server."
4. **Verify DNS is routing correctly** with a [DNS leak test](https://www.dnsleaktest.com/) (Standard test).

---

## 9. Docker on Standalone Ubuntu Server

**Outcome:** ✅ Successful install

```bash
# Update and upgrade
sudo apt update && sudo apt upgrade -y

# Add Docker's official GPG key
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the Docker apt repository
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

# Install Docker
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify
sudo systemctl status docker
```

---

## 10. Docker on Ubuntu Server Inside Proxmox

**Outcome:** ✅ Successful install

Same steps as [Project #9](#9-docker-on-standalone-ubuntu-server), run inside an Ubuntu Server VM hosted on Proxmox.

### Bonus: Run Kali Linux in a Docker container
```bash
sudo docker pull docker.io/kalilinux/kali-rolling
sudo docker run -it kalilinux/kali-rolling
apt update && apt upgrade -y
apt install -y sudo
```

### Alternative method: Docker + Portainer (tested and working)
```bash
# Update
sudo apt update && sudo apt upgrade -y

# Add Docker's GPG key and repo
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

# Install Docker
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Install Portainer (optional)
sudo docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```
Access Portainer at `http://[VM_IP]:9000` or `https://[VM_IP]:9443`.

---

## 11. VLAN + Linux Bridge Across Proxmox Nodes

**Outcome:** ✅ Successful install
**Goal:** Install Proxmox on a second node (`proxmoxsmall`, ThinkCentre M710s), set up a VLAN, and bridge it to the first node (ThinkCentre M920t, "LOCALHOST").

1. Install Proxmox VE on the second node (over an existing Ubuntu Server install).
2. Install sudo and update:
   ```bash
   apt install sudo
   sudo apt update && sudo apt upgrade -y
   ```
3. **On Server 2 (M710s), configure a second bridge.** Back up the interfaces file first:
   ```bash
   cp /etc/network/interfaces /etc/network/interfaces-default
   sudo nano /etc/network/interfaces
   ```
   Set it to:
   ```
   auto lo
   iface lo inet loopback

   iface nic0 inet manual
   iface nic1 inet manual

   auto vmbr0
   iface vmbr0 inet static
       address 192.168.1.9/24
       gateway 192.168.1.1
       bridge-ports nic0
       bridge-stp off
       bridge-fd 0
       bridge-vlan-aware yes
       bridge-vids 2-4094

   auto vmbr1
   iface vmbr1 inet static
       address 192.168.100.3/24
       gateway 192.168.100.1
       bridge-ports nic0.2
       bridge-stp off
       bridge-fd 0
       bridge-vlan-aware yes
       bridge-vids 2-4094

   source /etc/network/interfaces.d/*
   ```
   Save with `Ctrl+X` → **Yes**.
4. **On Server 1 (M920t), mirror the same change** (back up first, same as above), using its own IPs:
   ```
   auto lo
   iface lo inet loopback

   iface nic0 inet manual
   iface nic1 inet manual

   auto vmbr0
   iface vmbr0 inet static
       address 192.168.1.4/24
       gateway 192.168.1.1
       bridge-ports nic0
       bridge-stp off
       bridge-fd 0
       bridge-vlan-aware yes
       bridge-vids 2-4094

   auto vmbr1
   iface vmbr1 inet static
       address 192.168.100.2/24
       gateway 192.168.100.1
       bridge-ports nic0.2
       bridge-stp off
       bridge-fd 0
       bridge-vlan-aware yes
       bridge-vids 2-4094

   source /etc/network/interfaces.d/*
   ```
5. Restart networking on both nodes:
   ```bash
   sudo systemctl restart networking
   ```
6. Reboot both nodes, checking Datacenter → localhost → System → Network on each to confirm the bridge changes took effect.
7. **Create the cluster:** on Server 1's Proxmox UI, Datacenter → Cluster → **Create Cluster**, then **Join Information** to generate the join link/API token.
8. **Join the cluster:** on Server 2, Datacenter → Cluster → **Join Cluster**, and paste the info from Server 1.
9. Refresh both UIs to confirm the nodes are linked.

---

## 12. TrueNAS in Proxmox

**Outcome:** ✅ Successful install
**Host:** `proxmoxbeta` (ThinkCentre 920s)

1. Find the [community script](https://community-scripts.org/) for TrueNAS and run it from the Proxmox shell (Datacenter → proxmoxsmall → Shell):
   ```bash
   bash -c "$(curl -fsSL https://git.community-scripts.org/community-scripts/ProxmoxVE/raw/branch/main/vm/truenas-vm.sh)"
   ```
2. Accept the default install options; save storage to `proxmoxBetaExternal`; leave interface/settings at default.
3. Confirm the script completes successfully.
4. Let the initial backup/setup run, then complete first-run setup (takes a while).
5. Create the local admin account and log into the web UI.

---

## 13. Immich (Self-Hosted Photo Backup)

**Outcome:** ✅ Successful install and use
**Host:** `proxmoxsmall` (ThinkCentre 710s)
**Docs:** [docs.immich.app](https://docs.immich.app/overview/quick-start)

1. Run the [community script](https://community-scripts.org/) from the Proxmox shell:
   ```bash
   bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/immich.sh)"
   ```
2. Accept default install options, save to `proxmoxBetaExternal`, and leave Machine Learning options at default.
3. On success, the script prints the access URL, e.g. `http://<server-ip>:2283`.
4. Log into the web UI and create an account.
5. Install the Immich mobile app and sync it to the account.
6. Configure backup settings on the phone and verify a successful backup.

---

## 14. Pi-hole over Debian (proxmoxBeta)

**Outcome:** ✅ Successful install and deployment

1. Run the community LXC install script from the Proxmox shell (same pattern as other community-script installs above), accepting defaults and saving to `proxmoxBetaExternal`.
2. Set the Pi-hole password from the container shell:
   ```bash
   pihole setpassword
   ```
3. Log into the web UI.
4. Configure DNS.
5. Configure DHCP — **set up Pi-hole's DHCP before the router's**, matching the ordering note from [Project #8](#8-pi-hole-in-proxmox-ubuntu-24-lts--network-wide-dns).
6. Review logs to confirm the DHCP handoff worked.
7. Import additional blocklists.

---

## 15. Metasploitable2 in Proxmox

**Outcome:** ✅ Successful install and deployment
**Goal:** Stand up a deliberately vulnerable target for penetration-testing practice.

1. Deploy an Ubuntu LXC via the [community script](https://community-scripts.org/scripts/ubuntu):
   ```bash
   bash -c "$(curl -fsSL https://git.community-scripts.org/community-scripts/ProxmoxVE/raw/branch/main/ct/ubuntu.sh)"
   ```
2. From the Ubuntu shell, download and convert the Metasploitable2 disk image:
   ```bash
   wget https://sourceforge.net/projects/metasploitable/files/Metasploitable2/metasploitable-linux-2.0.0.zip
   unzip metasploitable-linux-2.0.0.zip
   cd Metasploitable2-Linux/
   qemu-img convert -O qcow2 Metasploitable.vmdk metasploitable.qcow2
   qm create <VM_ID> --memory 2048 --cores 2 --name Metasploitable2 --net0 virtio,bridge=vmbr0 --boot c --bootdisk ide0
   qm importdisk <VM_ID> metasploitable.qcow2 local-lvm
   qm set <VM_ID> --ide0 <LOCAL_STORAGE_NAME>:vm-<VM_ID>-disk-0
   ```
   Replace `<VM_ID>` and `<LOCAL_STORAGE_NAME>` with your actual VM ID and storage pool name.

> ⚠️ Run Metasploitable only on an isolated lab network — it's intentionally full of vulnerabilities.

---

## 16. Wazuh SIEM

**Outcome:** ✅ Successful install and deployment
**Host:** `proxmoxBeta` (ThinkCentre 920s)
**Docs:** [documentation.wazuh.com](https://documentation.wazuh.com/current/installation-guide/index.html)

1. Run the [community script](https://community-scripts.org/) from the Proxmox shell:
   ```bash
   bash -c "$(curl -fsSL https://git.community-scripts.org/community-scripts/ProxmoxVE/raw/branch/main/ct/wazuh.sh)"
   ```
2. Accept defaults, save to `proxmoxBetaExternal`.
3. Retrieve the generated credentials from the container shell:
   ```bash
   cat ~/wazuh.creds
   ```
4. Log into the web UI.

---

## 17. Splunk SIEM

**Outcome:** ✅ Successful install and deployment
**Host:** `proxmoxBeta` (ThinkCentre 920s)
**Docs:** [community-scripts.org/scripts?q=splunk](https://community-scripts.org/scripts?q=splunk)

1. Run the [community script](https://community-scripts.org/) from the Proxmox shell:
   ```bash
   bash -c "$(curl -fsSL https://git.community-scripts.org/community-scripts/ProxmoxVE/raw/branch/main/ct/splunk-enterprise.sh)"
   ```
2. Accept defaults, save to `proxmoxBetaExternal`.
3. Recover credentials from the container:
   ```bash
   sudo nano splunk.creds
   ```
4. Log into the web UI.

---

## 18. ITSM-NG (Asset / Budget / Ticketing)

**Outcome:** ✅ Successful install and deployment
**Host:** `proxmoxBeta` (ThinkCentre 920s)
**Docs:** [community-scripts.org/scripts/itsm-ng](https://community-scripts.org/scripts/itsm-ng)

1. Run the [community script](https://community-scripts.org/) from the Proxmox shell:
   ```bash
   bash -c "$(curl -fsSL https://git.community-scripts.org/community-scripts/ProxmoxVE/raw/branch/main/ct/itsm-ng.sh)"
   ```
2. Accept defaults, save to `proxmoxBetaExternal`.
3. Default credentials: `itsm` / `itsm` — **change these immediately after first login.**
4. Log into the web UI.
5. Create a database in MariaDB, add a dedicated DB user, and grant privileges.
6. Populate the inventory list and add sample tickets to validate the setup.

---

## 19. SnipeIT (Asset / Budget / Ticketing)

**Outcome:** ✅ Successful install and deployment
**Host:** `proxmoxBeta` (ThinkCentre 920s)
**Docs:** [community-scripts.org/scripts/snipe](https://community-scripts.org/scripts/snipe) · [CLI utilities reference](https://snipe-it.readme.io/docs/command-line-utilities)

1. Run the [community script](https://community-scripts.org/) from the Proxmox shell:
   ```bash
   bash -c "$(curl -fsSL https://git.community-scripts.org/community-scripts/ProxmoxVE/raw/branch/main/ct/snipeit.sh)"
   ```
2. Accept defaults, save to `proxmoxBetaExternal`.
3. Review the [post-install CLI utilities doc](https://snipe-it.readme.io/docs/command-line-utilities).
4. Log into the web UI.
5. Populate the inventory list and add sample tickets.

---

## Quick Reference & Troubleshooting Notes

### Network testing
```bash
# Test local interface
ping -c 4 192.168.1.100

# Test gateway
ping -c 4 192.168.1.1

# Test external connectivity
ping -c 4 8.8.8.8

# Test DNS resolution
dnslookup google.com
```

### System info tool
[`fastfetch`](https://github.com/fastfetch-cli/fastfetch) — a `neofetch` replacement for readable system info at a glance:
```bash
sudo apt install fastfetch
```

### Changing a Proxmox host's IP manually
1. Access the Proxmox host via SSH or console.
2. Edit `/etc/network/interfaces` (e.g. with `nano` or `vim`).
3. Update the address, gateway, and netmask as needed for `vmbr0`:
   ```
   auto vmbr0
   iface vmbr0 inet static
       address 192.168.1.4/24
       gateway 192.168.1.1
       bridge-ports eth0
       bridge-stp off
       bridge-fd 0
   ```

### Switching Proxmox from the Enterprise repo to no-subscription
1. Log into the Proxmox web UI.
2. Go to **Updates → Repositories**.
3. Disable the Enterprise repository (do this for both the main repo and the Ceph repo before running updates).
4. Add the No-Subscription repository via **Add**.
5. Update packages:
   ```bash
   sudo apt update && apt upgrade
   ```

> **Lesson learned:** setting up a VLAN on the localhost node once broke IP addressing on Proxmox entirely. Recovery required manually editing `/etc/network/interfaces` to get the host back on the same network as the rest of the devices. Back up your interfaces file before any network changes (see Project #11 for the pattern used since).

### Rename a Konsole terminal tab
```bash
echo -ne "\033]30;SmallTower\007"
```

### Proxmox ISO storage location
ISO files for Proxmox go in `/var/lib/vz/template/iso` on the Proxmox host.

### Disable the system beep
```bash
sudo modprobe --remove pcspkr
```

### Wazuh: "Invalid server address found: 'MANAGER_IP'"
**Symptom** (seen on Ubuntu 20.04):
```
wazuh-agentd: ERROR: (4112): Invalid server address found: 'MANAGER_IP'
wazuh-agentd: CRITICAL: (1215): No client configured. Exiting.
wazuh-agentd did not start
```
**Cause:** the `server.address` field in `/var/ossec/etc/ossec.conf` doesn't contain a valid IP.

**Fix:**
```bash
sudo nano /var/ossec/etc/ossec.conf
```
Set the `<address>` field under `<server>` to your actual manager IP, e.g.:
```xml
<client>
  <server>
    <address>127.0.0.1</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
  <config-profile>ubuntu, ubuntu20, ubuntu20.04</config-profile>
  <notify_time>10</notify_time>
  <time-reconnect>60</time-reconnect>
  <auto_restart>yes</auto_restart>
  <crypto_method>aes</crypto_method>
</client>
```

### Install Docker on Ubuntu 24.04 (inside a Proxmox VM)
```bash
# Update packages and install prerequisites
sudo apt update
sudo apt upgrade -y
sudo apt install -y ca-certificates curl gnupg lsb-release

# Add Docker's GPG key and repository
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Install Docker Engine, CLI, and containerd
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start and enable Docker
sudo systemctl enable --now docker

# Verify
sudo docker run --rm hello-world
docker --version
```

---

## Links of Note

| Resource | Description |
|---|---|
| [Proxmox Docs — Network Configuration](https://docs.bankai-tech.com/Proxmox/Docs/Networking/Network%20Configuration/) | Proxmox networking reference |
| [Proxmox VE Administration Guide](https://pve.proxmox.com/pve-docs/pve-admin-guide.html) | Official Proxmox admin guide |
| [Windows VirtIO Drivers](https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers) | VirtIO driver install guide for Windows VMs |
| [Downloading Ubuntu Server ISO](https://linuxvox.com/blog/download-ubuntu-server-iso/) | Comprehensive ISO download guide |
| [Pi-hole Basic Install](https://docs.pi-hole.net/main/basic-install/) | Official Pi-hole install docs |
| [Kali in Docker walkthrough](https://th4ntis.com/guide/2024/03/11/Kali-Docker.html) | Kali Linux Docker install guide |
| [Kali in Proxmox LXC walkthrough](https://benheater.com/proxmox-kali-linux-container-lxc/) | Kali Linux Proxmox LXC guide |
| [Kali LXC images](https://images.linuxcontainers.org/images/kali/current/amd64/default/) | Official Kali container images |
| [Proxmox VLAN configuration guide](https://medium.com/@P0w3rChi3f/proxmox-vlan-configuration-a-step-by-step-guide-edc838cc62d8) | Step-by-step VLAN walkthrough |
| [OpenVAS Docker image](https://hub.docker.com/r/mikesplain/openvas) | OpenVAS install directions |

---

## Skills Demonstrated

- CLI, Bash, and shell scripting
- SMB setup and implementation
- Self-hosted image/photo backup server (Immich, as a Google Photos alternative)
- Home Assistant installation, IoT device inventory, and asset configuration
- ITSM-NG for inventory management, ticketing, and budget tracking
- Wazuh for SIEM and security event reporting
- Database work with MariaDB, MySQL, and MongoDB (backing SnipeIT, ITSM-NG)
- Offensive security tooling: Metasploitable, msfconsole/Metasploit, running exploits and shells against lab targets
- Broad Linux distro experience: Parrot, Debian, Ubuntu, Lubuntu, Zorin, Kali, Arch, Mint, Ubuntu Server
- Windows OS experience across XP, 7, 8, 10, and 11, including using ESD and Hiren's BootCD to recover systems from faults
- Windows Defender, Azure, Active Directory, Windows Server 2022, and PowerShell
- Virtualization and containerization with Proxmox, Docker, Portainer, and VirtualBox
