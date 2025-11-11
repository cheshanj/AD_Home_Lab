
# 🛠️ Active Directory Home Lab – VirtualBox Deployment

In this lab, I have set up several virtual machines in VirtualBox to simulate a real-world Active Directory environment. The lab is inspired by several online tutorials. I have listed them at the [end of the report](https://github.com/cheshanj/AD_Home_Lab/blob/main/docs/Active_Directory_Home_Lab.md#1%EF%B8%8F%E2%83%A36%EF%B8%8F%E2%83%A3-resources).

- [Video demonstration](https://www.youtube.com/watch?v=EPgwXccCs2Y)
- [Full report (Still updating..)](/Set%20up%20an%20Active%20Directory%20home%20lab.docx) 

---

## 1️⃣ Prerequisites

- VirtualBox or VMware Workstation
- Windows Server 2019, 2022, or 2025 ISO. I have used the Server 2022 Desktop Experience Evaluation in this lab.
- Windows 10 ISO

---

## 2️⃣ Installing VirtualBox

Download the latest stable [VirtualBox](https://www.virtualbox.org/wiki/Downloads) x64 version. Then install the software with default settings. You may also need to download the Extension Pack to enable guest services on client VMs.

---

## 3️⃣ Downloading Windows ISOs

You can download all required ISOs from the [Microsoft website](https://www.microsoft.com/en-ca/software-download). To download the correct version, search for the required operating system, and the search results will direct you to the download page.

---

## 4️⃣ Create the Virtual Machines (VMs)

### ✅ 4.1 Create the Server VM
1. Run VirtualBox
2. Click **New**
3. Set a name for the server
4. Select storage location
5. Leave ISO location unchanged (VM template first)
6. OS type → **Microsoft Windows**
7. Version → **Windows Server 2022 (64-bit)**
8. Keep default hardware + hard disk settings
9. Click **Finish**

### ✅ 4.2 Create the Client VM
1. Same general steps as server setup
2. OS Version → **Windows 10 (64-bit)**
3. Click **Finish**

### ✅ 4.3 Create a VM Group *(optional but recommended)*

Helps keep VMs organized inside VirtualBox.

---

## 5️⃣ Update VM Settings

1. Right-click VM → **Settings**
2. Go to **General → Advanced**
3. Enable:
   - **Shared Clipboard → Bidirectional**
   - **Drag & Drop → Bidirectional**
4. Set Boot Order:
   - First: **Optical**
   - Second: **Hard Disk**
5. Networking:

### 🌐 Server Network Setup
- Adapter 1 → **NAT**
- Adapter 2 → **Internal Network**
- Rename Internal Network → **VMInternal**

### 💻 Client Network Setup
- Adapter 1 → **Internal Network**
- Network Name → **VMInternal**

Other settings remain default unless customization is needed.

---

## 6️⃣ Install Operating Systems

1. Start VM
2. When prompted, mount correct ISO
3. Run installation wizard

✅ **Server OS Selection:**  
- Windows Server 2022 Evaluation Desktop Experience

✅ **Client OS Selection:**  
- Windows 10 Pro (or Education — must support domain join)

After installation:
- Server → Set password for local Administrator  
- Client → Create any username/password  

---

## 7️⃣ Install Guest Additions

1. Open File Explorer (This PC)
2. VirtualBox menu → **Devices → Insert Guest Additions CD**
3. Install and reboot

---

## 8️⃣ Rename Each Machine

- Example: Server → **DC01**, Client → **VMC01**
- Can be done via **Settings → Computer Details** or **sysdm.cpl**
- Reboot after rename

---

## 9️⃣ Configure Network Settings

### 🔧 Server Internal Adapter
- IP: **172.16.0.1**
- Subnet: **255.255.255.0**
- DNS: **127.0.0.1**

### 🔧 Client
- Only Internal Network (VMInternal)
- DHCP from DC will assign IP automatically

---

## 🔟 Install Active Directory Domain Services (AD DS)

1. Open **Server Manager**
2. Click **Add Roles and Features**
3. Choose **Role-based installation**
4. Select server and enable **Active Directory Domain Services**
5. Complete wizard
6. Notification appears → **Promote this server to Domain Controller**

Since no existing domain/forest:
- Create **New Forest**
- Enter Root Domain Name (FQDN) → e.g., `company1.com`
- Set DSRM password
- Wizard checks prerequisites
- Promote server and reboot

After reboot:
- Local Administrator disabled
- Use **Domain Administrator** going forward

---

## 1️⃣1️⃣ Create a Domain Administrator Account

1. Open **Active Directory Users and Computers**
2. Create new **OU** named `_ADMINS`
   - Uncheck **Protect from accidental deletion** *(lab only)*
3. Create new user under `_ADMINS`
4. Set password
5. Uncheck all boxes except **Password never expires** *(lab only)*
6. Open **Properties → Member Of**
7. Add **Domain Admins**
8. Log in using new domain admin account

---

## 1️⃣2️⃣ Install RAS & NAT (Internet Access for Clients)

Clients are internal-only and need the server as a gateway.

### ✅ RAS Installation
1. Add Roles and Features
2. Select **Remote Access**
3. Enable **Routing**
4. Complete wizard

### ✅ NAT Configuration
1. Open **Routing and Remote Access**
2. Right-click server → **Configure and Enable**
3. Select **NAT**
4. Choose external interface → NAT adapter
5. Finish setup
6. Server status should display **UP**

---

## 1️⃣3️⃣ Configure DHCP

### ✅ Installation
1. Add Roles and Features
2. Select **DHCP Server**
3. Finish wizard

### ✅ DHCP Scope Setup
1. Open DHCP Console
2. Right-click **IPv4 → New Scope**
3. IP Range:
   - **172.16.0.100 → 172.16.0.200**
   - Mask: **255.255.255.0**
4. Router: **172.16.0.1**
5. DNS: **172.16.0.1**
6. Activate scope

---

## 1️⃣4️⃣ Connect Windows 10 Client to the Domain

1. Run → `sysdm.cpl`
2. Click **Change**
3. Join Domain → enter root domain
4. Enter domain admin credentials
5. Restart client
6. Verify networking:

✅ `ipconfig /all` – receives DHCP address  
✅ `ping 172.16.0.1` – connectivity to DC  
✅ `nslookup DOMAINNAME` – DNS functioning  
✅ `ping www.google.com` – internet through NAT  

---

## 1️⃣5️⃣ Verify Server-Side

- On DC → open DHCP console
- Check **Address Leases**
- Client should appear with assigned IP ✅

---

## 1️⃣6️⃣ Resources

- [Building a Virtual Security Home Lab: Part 6 - Active Directory Lab Setup - Part 1 | Source Code](https://blog.davidvarghese.net/posts/building-home-lab-part-6/)
- [How to Setup a Basic Home Lab Running Active Directory (Oracle VirtualBox) | Add Users w/PowerShell](https://www.youtube.com/watch?v=MHsI8hJmggI&list=PLqBeiU46hx1H--SNfTrohTOWeqkK-M2Y0)
- [Home Lab Beginners guide (Hardware)](https://linuxblog.io/home-lab-beginners-guide-hardware/)
- [Building a Security Lab in VirtualBox](https://benheater.com/building-a-security-lab-in-virtualbox/)
- [r/Homelab](https://www.reddit.com/r/homelab/)


---

## ✅ Lab Practices vs Real-World Best Practices
> [!IMPORTANT]
> Most of the lab settings were used to keep the setup simple.
> But when it comes to the real world, always follow the best practice or company policies & guidelines.

| Setting | Lab Choice | Real-World Best Practice |
|--------|------------|--------------------------|
| Password never expires | Enabled | Should be disabled — enforce password aging |
| OU accidental deletion unchecked | Unchecked | Keep checked to protect objects |
| Only one Domain Controller | Yes | Minimum two DCs for redundancy |
| DC hosts DHCP + NAT | Yes | Normally hosted on separate servers/firewalls |

---
