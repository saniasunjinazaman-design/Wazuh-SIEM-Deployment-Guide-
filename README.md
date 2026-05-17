# Wazuh SIEM Deployment Guide

> A step-by-step guide to provisioning an Ubuntu 22.04 virtual machine, deploying the Wazuh security platform, and accessing the environment remotely via PuTTY.
> 

---

## 📋 Table of Contents

- [Prerequisites](https://www.notion.so/363b25979ed08042b27acf4b98213622?pvs=21)
- [Part 1 — Setting Up Ubuntu 22.04 in a VM](https://www.notion.so/363b25979ed08042b27acf4b98213622?pvs=21)
- [Part 2 — Initial Ubuntu Configuration](https://www.notion.so/363b25979ed08042b27acf4b98213622?pvs=21)
- [Part 3 — Deploying Wazuh](https://www.notion.so/363b25979ed08042b27acf4b98213622?pvs=21)
- [Part 4 — Remote Access via PuTTY](https://www.notion.so/363b25979ed08042b27acf4b98213622?pvs=21)

---

## Prerequisites

Before you begin, ensure you have the following:

| Requirement | Details |
| --- | --- |
| Hypervisor | VirtualBox 7.x or VMware Workstation 17+ |
| Ubuntu ISO | [Ubuntu 22.04.x LTS (Jammy Jellyfish)](https://ubuntu.com/download/server) |
| Host RAM | Minimum **8 GB** (16 GB recommended) |
| Disk Space | Minimum **50 GB** free on host |
| PuTTY | [Download PuTTY](https://www.putty.org/) (Windows) |
| Internet | Active connection required |

> [!NOTE]
This guide uses **VirtualBox** as the hypervisor. Steps may differ slightly for VMware.
> 

---

## Part 1 — Setting Up Ubuntu 22.04 in a VM

### 1.1 Create a New Virtual Machine

1. Open **VirtualBox** and click **New**.
2. Configure the VM with the following settings:
    
    
    | Setting | Value |
    | --- | --- |
    | Name | `Ubuntu-22.04-Wazuh` |
    | Type | `Linux` |
    | Version | `Ubuntu (64-bit)` |
    | Base Memory | `4096 MB` (4 GB minimum) |
    | Processors | `2 CPUs` minimum |
    | Virtual Hard Disk | `50 GB` (dynamically allocated) |
3. Click **Finish** to create the VM.

### 1.2 Attach the Ubuntu ISO

1. Select your new VM and click **Settings → Storage**.
2. Under **Controller: IDE**, click the **Empty** optical drive.
3. Click the disc icon on the right → **Choose a disk file…**
4. Browse to your downloaded `ubuntu-22.04-live-server-amd64.iso` and click **Open**.
5. Click **OK** to save.

### 1.3 Configure Network Adapter

1. Go to **Settings → Network → Adapter 1**.
2. Set **Attached to:** `Bridged Adapter`.
3. Select your host's active network interface from the **Name** dropdown.
4. Click **OK**.

> [!TIP]
**Bridged Adapter** places your VM on the same network as your host, which is required for PuTTY remote access.
> 

### 1.4 Install Ubuntu 22.04

1. Start the VM by clicking **Start**.
2. Select **Try or Install Ubuntu Server** and press `Enter`.
3. Follow the installer prompts:
    - **Language:** English
    - **Keyboard Layout:** Your preference
    - **Network:** Confirm your NIC is detected (note the assigned IP address)
    - **Storage:** Use the entire disk (default)
    - **Profile Setup:**

     `Your name:        <your full name> 
     Server name:      wazuh-server      
     Username:         <your username>   
     Password:         <strong password>` 

- **SSH Setup:** ✅ Check **Install OpenSSH server** — required for PuTTY access
- **Featured Snaps:** Skip (press `Done`)
1. Wait for installation to complete, then select **Reboot Now**.
2. When prompted, remove the ISO: **Devices → Optical Drives → Remove disk from virtual drive**, then press `Enter`.

---

## Part 2 — Initial Ubuntu Configuration

After the VM reboots, log in with your credentials and run the following commands.

### 2.1 Update the System

bash

`sudo apt update && sudo apt upgrade -y`

### 2.2 Note Your IP Address

bash

`ip a`

Look for your IP under the active interface (e.g., `enp0s3`). It will look like `192.168.x.x`.

> [!IMPORTANT]
**Save this IP address** — you will need it in [Part 4](https://www.notion.so/363b25979ed08042b27acf4b98213622?pvs=21) for PuTTY and for accessing the Wazuh dashboard.
> 

### 2.3 Verify SSH is Running

bash

`sudo systemctl status ssh`

The output should show `active (running)`. If not, start it:

bash

`sudo systemctl enable --now ssh`

<img width="790" height="340" alt="image" src="https://github.com/user-attachments/assets/4f08a567-0abf-452e-8d78-60d9fce17d2d" />


---

## Part 3 — Deploying Wazuh

Wazuh is deployed using the **all-in-one** installer, which sets up the Wazuh Manager, Indexer, and Dashboard on a single node.

### 3.1 Download the Installer

bash

`curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh`

### 3.2 Run the All-in-One Installation

bash

`sudo bash ./wazuh-install.sh -a`

> [!NOTE]
This process typically takes **10–20 minutes** depending on your system speed and internet connection. Do not interrupt it.
> 

<img width="696" height="83" alt="image" src="https://github.com/user-attachments/assets/83b0dc76-4acb-4cec-871a-e629cb567d88" />


### 3.3 Retrieve Dashboard Credentials

At the end of the installation, the installer prints the auto-generated credentials. Look for output similar to:

<aside>

> INFO: --- Summary ---
INFO: You can access the web interface https://<your-ip>
    User: admin
    Password: <auto-generated-password>
> 
</aside>


> [!IMPORTANT]
**Copy and save these credentials immediately.** If you miss them, extract them with:
> 
> 
> bash
> 
> <aside>
> 
> 
> `sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt`
> <img width="543" height="124" alt="Screenshot 2026-05-16 212340" src="https://github.com/user-attachments/assets/64447ebc-817a-416c-bd1e-4edf44bc765e" />

> </aside>
> 

### 3.4 Access the Wazuh Dashboard

1. On your host machine, open a browser.
2. Navigate to:

> `https://<your-vm-ip>`
> 
1. Accept the self-signed certificate warning.
2. Log in with the credentials from Step 3.3.

You should now see the **Wazuh Security Dashboard**.

<img width="1912" height="1032" alt="image" src="https://github.com/user-attachments/assets/9d4f5c44-811e-44eb-be7c-e4f375ac2435" />


---

## Part 4 — Remote Access via PuTTY

PuTTY allows you to manage your Ubuntu VM over SSH from your Windows host.

### 4.1 Install PuTTY

Download and install PuTTY from the official site:
👉 https://www.putty.org/

### 4.2 Connect to Your VM

1. Open **PuTTY**.
2. In the **Session** panel, configure the following:

| Field | Value |
| --- | --- |
| Host Name (or IP address) | `<your-vm-ip>` (from Step 2.2) |
| Port | `22` |
| Connection type | `SSH`  |

<img width="593" height="537" alt="image" src="https://github.com/user-attachments/assets/3deb7411-2592-41a9-a1e9-9c978351fb25" />


1. *(Optional)* In the **Saved Sessions** field, type `Wazuh-Ubuntu`, then click **Save** for future use.
2. Click **Open**.
3. Accept the host key fingerprint by clicking **Accept**.
4. Log in with your Ubuntu username and password.

You are now connected to your Ubuntu VM remotely via SSH.

<img width="791" height="743" alt="image" src="https://github.com/user-attachments/assets/b8595a01-e7bd-4c25-bd0e-9566ecee8f33" />


### 4.3 Keep the Session Alive (Recommended)

To prevent PuTTY from timing out during idle periods:

1. Go to **Connection** in the left panel.
2. Set **Seconds between keepalives** to `30`.
3. Return to **Session** and click **Save** to update your saved session.
