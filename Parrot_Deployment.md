## 🛠️ Lab: Deploying Parrot OS in a Virtual Machine

This lab is designed for **KVM/QEMU** (using `virt-manager`), as it offers excellent performance on Linux hosts.

### I. Prerequisites & Preparation

Before creating the VM, you need the installation media.

1. **Download:** Head to the [Parrot OS Download page](https://parrotsec.org/download/) and grab the **Security Edition** (ISO).

2. **Verification:** Always verify your ISO hash to ensure the download wasn't corrupted or tampered with.

Bash

    ```
    sha256sum Parrot-security-6.x_x64.iso
    ```
    

### II. Creating the VM Container

1. Open **Virtual Machine Manager** (`virt-manager`).

2. Click **"New Virtual Machine"** (the monitor icon).

3. Select **Local install media (ISO image)**.

4. Browse and select your Parrot ISO.

- _Note:_ If it doesn't auto-detect, search for "Debian 12" (or the current Parrot base).

5. **Resources:** * **Memory:** At least **4096 MB** (4GB).

	- **CPUs:** At least **2**.

6. **Storage:** Allocate at least **40 GB**. Parrot comes pre-loaded with many tools, so it needs more space than a "vanilla" distro.
    
7. **Final Check:** Name it `Parrot-Security-Lab` and ensure **"Customize configuration before install"** is checked if you want to use **VirtIO** for better disk/network performance.
    

---

### III. The Installation Process

1. **Boot:** Choose "Try/Install" from the GRUB menu.
    
2. **Launch Installer:** Once the Live Desktop loads, click the **Install Parrot** icon.
    
3. **Partitioning:** For a lab environment, **"Erase disk"** is the simplest route.
    
    - _Pro-Tip:_ If you plan on doing advanced forensics later, consider a separate `/home` partition so you can reinstall the OS without losing your "loot" or reports.
        
4. **User Setup:** Create your user account.
    
    > [!IMPORTANT]
    > 
    > **Secure by Default:** Parrot does not give the user `root` permissions by default. You will use `sudo` for administrative tasks.
    
![[Pasted image 20260501225200.png]]
---

### IV. Post-Install Optimization (The "Obsidian Checklist")

Copy this into your vault to track your new VM's setup:

- [ ] **Update System:** Run `sudo parrot-upgrade` (Parrot uses this wrapper for `apt full-upgrade`).
    
- [ ] **Install Guest Tools:** Ensure `spice-vdagent` and `qemu-guest-agent` are running for shared clipboards and window resizing.
    
- [ ] **Snapshot:** Once fully updated and configured, **Power Off** and take a Snapshot named `Clean_Install`.
    
- [ ] **Configure Shared Folders:** Set up a shared directory between your host and the VM for moving log files or scripts easily.
    

---

### V. Why Parrot over Kali?

|**Feature**|**Parrot OS**|**Kali Linux**|
|---|---|---|
|**User Focus**|Security + Privacy + Development|Pure Pentesting|
|**Performance**|Uses MATE (very lightweight)|Uses XFCE (lightweight but Parrot feels snappier)|
|**Default Tools**|Pentesting tools + Anonsurf (Privacy)|Massive library of Pentesting tools|
|**Stability**|Known for being a solid daily driver|Can be "brittle" if used as a primary OS|