# Lab: Setting Up a Windows VM on Pop!_OS (KVM/QEMU)

**Tools:** Virt-Manager, KVM, QEMU **Goal:** Create a high-performance Windows environment for networking labs.

---

## 1. Prerequisites (The Pop!_OS Setup)

Before opening any software, you need to ensure the virtualization stack is installed and your user has permission to use it.

Open your terminal and run: `sudo apt update && sudo apt install virt-manager qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virtinst libvirt-daemon`

Then, add your user to the `libvirt` and `kvm` groups so you don't have to type your password every time: `sudo adduser $USER libvirt` `sudo adduser $USER kvm`

> [!IMPORTANT] **Reboot your computer** after running these commands to apply the group changes.

---

## 2. Download the Essentials

1. **Windows ISO:** Download the Media Creation Tool or ISO from Microsoft's site.
    
2. **VirtIO Drivers:** This is the "secret sauce" for Linux VMs. Windows doesn't natively know how to talk to KVM's fast virtual hardware.
    
    - Download the **virtio-win.iso** from the Fedora Project (stable version).
        

---

## 3. Creating the VM

1. Open **Virtual Machine Manager** from your applications.
    
2. Click **File > New Virtual Machine**.
    
3. Choose **Local install media (ISO image)**.
    
4. Browse and select your Windows ISO.
    
5. **Resources:** - Give it at least **8GB RAM** (8192MB) and **4 CPU cores** if your hardware allows.
    
    - Allocate at least **60GB** for the virtual disk.
        
6. **Crucial Step:** Before clicking "Finish," check the box **"Customize configuration before install."**
    

---

## 4. The "Performance" Configuration

In the customization screen, change these settings for the best experience:

- **CPUs:** Set "Model" to `host-passthrough`. This lets Windows see your actual CPU features.
    
- **Storage (Disk):** Change "Disk bus" to **VirtIO**.
    
- **Network (NIC):** Change "Device model" to **virtio**.
    
- **Display/Video:** Use **Virtio** or **QXL**.
    
- **Add Hardware:** Click "Add Hardware" -> "Storage" -> Select the **virtio-win.iso** you downloaded earlier. Set its device type to **CDROM**.
    

---

## 5. The Installation Trick

When Windows asks "Where do you want to install Windows?" the list will be **empty**. Don't panic!

1. Click **Load Driver**.
    
2. Browse to the VirtIO CD-ROM you attached.
    
3. Look for the folder: `amd64 > w11` (or `w10`).
    
4. Select the **viostor** driver.
    
5. Once loaded, your hard drive will magically appear. Finish the install as normal.
    

---

## 6. Post-Install Cleanup

Once you're at the Windows desktop:

1. Open "This PC" and open the VirtIO CD-ROM.
    
2. Run `virtio-win-gt-x64.exe`. This installs the drivers for your network and smooth mouse movement.
    
3. In Virt-Manager, go to **View > Resize to VM** to get a perfect resolution.
    

---

## Lab Analysis (For your Notebook)

- **Performance Check:** Open Task Manager in Windows. Does the CPU name match your actual hardware? (This confirms `host-passthrough` worked).
    Pass-through confirmed 
    
- **Networking:** Can the VM ping your Pop!_OS host? This is the foundation for your future routing labs.

