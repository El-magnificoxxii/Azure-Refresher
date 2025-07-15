# Windows VM Web Server Deployment on Azure

This project demonstrates deploying a Windows Virtual Machine (VM) on Azure to host **two IIS-based websites**. It covers two major phases:

1. Hosting websites using **port-based access** (e.g., port 80 and 8080)
2. Transitioning to **host-based routing** using **Azure Application Gateway** and **custom DNS** from DuckDNS.
3. Enabling **HTTPS using Let's Encrypt** certificates installed via **Win-ACME** and bound to **Azure Application Gateway**

---

## 🚀 Phase 1: Hosting Two Websites with IIS and NSG Rules


### 🌐 1. Create Virtual Network (VNet)

| Setting              | Value                 |
|----------------------|-----------------------|
| **Name**             | `webapp-test-vnet`    |
| **Resource Group**   | `webapp-test-rg`      |
| **Region**           | `UK West`             |
| **Address Space**    | `10.0.0.0/22`         |
| **Subnet**           | `default (10.0.0.0/24)` |
| **Azure Bastion**    | Disabled              |
| **Azure Firewall**   | Disabled              |
| **Azure DDoS**       | Disabled              |

---

### 🔐 2. Create Application Security Groups (ASG)

| Name        | Region   | Resource Group     |
|-------------|----------|--------------------|
| win-vm-asg  | UK West  | webapp-test-rg     |
| lin-vm-asg  | UK West  | webapp-test-rg     |

---

### 💻 3. Create Windows Virtual Machine

| Setting                     | Value                         |
|-----------------------------|-------------------------------|
| **Name**                    | `win-vm`                      |
| **Region**                  | UK West                       |
| **Image**                   | Windows Server 2022 Datacenter (Gen2) |
| **Size**                    | Standard D2s v3 (2 vCPUs, 8 GB RAM) |
| **Username**                | Abdullateef                   |
| **Public IP**               | win-vm-ip                     |
| **Inbound Ports**           | RDP, HTTP, HTTPS              |
| **VNet/Subnet**             | `webapp-test-vnet / default`  |
| **Accelerated Networking**  | Enabled                       |
| **OS Disk Type**            | Standard HDD LRS              |
| **Boot Diagnostics**        | Enabled                       |
| **Backup / Site Recovery**  | Disabled                      |
| **Monitoring / Alerts**     | Disabled                      |

---

### 🛠️ 4. Configure Web Server (IIS)

#### A. Connect to the VM

1. Download the RDP file from the Azure portal.
2. Log in using the credentials set during deployment.

#### B. Install IIS via PowerShell

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

![](./Assets/installingserv.png)

#### C. Confirm IIS Is Running

1. Open a browser on your local machine.
2. Navigate to your VM’s **public IP**.
3. You should see the default IIS landing page.

#### D. Deploy Your First Website

1. Go to `C:\inetpub\wwwroot`
2. Remove the default files (including `iisstart.htm`)
3. Paste your own website files into the folder

✅ You should now be able to access your first website via:

```
http://<VM_Public_IP>
```

![](./Assets/website1.png)

---

### 🌐 5. Host a Second Website on the Same VM (Different Port)

#### A. Create Folder for Site 2

1. Navigate to `C:\inetpub\`
2. Create a new folder: `site2`
3. Paste the second website’s files into `site2`

#### B. Configure Second Website in IIS

1. Open **IIS Manager**
2. In the left panel, right-click **Sites** > **Add Website**
   - **Site Name**: `Site2`
   - **Physical Path**: `C:\inetpub\site2`
   - **Binding**:
     - **IP Address**: All Unassigned
     - **Port**: `8080`
     - **Host name**: *(leave blank)*
3. Click **OK** to create the site

![](./Assets/iismanagerwebsite2.png)

---

### 🔒 6. Update NSG to Allow Port 8080

Create or update a rule in the NSG associated with your VM to allow traffic on port 8080.

| Field       | Value    |
|-------------|----------|
| **Source**  | Any      |
| **Destination** | Any |
| **Port**    | 8080     |
| **Protocol**| TCP      |
| **Action**  | Allow    |
| **Priority**| 200      |

✅ You should now be able to access your second website via:

```
http://<VM_Public_IP>:8080
```

![](./Assets/website2.png)

---
## 📌 Summary for Phase 1

- VM hosts two websites on ports `80` and `8080`
- IIS used for configuration and managing web application
- NSG and ASG applied for network control
- Sites accessible publicly with correct ports

---
## 🌐 Phase 2: Use DNS + Azure Application Gateway (No More Ports!)

### 🌍 Adding DNS to Two Websites Using DuckDNS

1. Head to a domain provider — this project uses [DuckDNS](https://www.duckdns.org/), a free dynamic DNS service.
2. Created two public domain names:
   - `reasonablecars.duckdns.org`
   - `tourchboxz.duckdns.org`
3. Updated both domains to point to the **same VM Public IP** (since both sites run on the same machine).
   - At this stage, **ports were not included** in the DNS record.
   - However, Site 2 required manual port access (`:8080`) via browser.

---

### 🧪 Interim DNS Testing with DuckDNS

Before configuring Azure Application Gateway, both websites were temporarily accessible using DuckDNS public DNS records:

- Site 1: `reasonablecars.duckdns.org` pointed to VM’s public IP (port 80)
- Site 2: `tourchboxz.duckdns.org` pointed to VM’s public IP (port 8080)

✅ Result:
- You could access Site 1 directly via `http://reasonablecars.duckdns.org`
- You could access Site 2 only by appending the port manually: `http://tourchboxz.duckdns.org:8080`

![](./Assets/cars80withoutagw.png)
![](./Assets/tourch8080withoutagw.png)
This worked, but it was not user-friendly or scalable.

➡️ Solution: Move to host-based routing with Azure Application Gateway and remove the need for port-specific access.


### 2️⃣ Both sites are reconfigured to share port 80, but are distinguished by hostnames:

- Changed `tourchboxz.duckdns.org` site in IIS to use:
  - **Port**: `80`
  - **Host Name**:
      - Site 2: `tourchboxz.duckdns.org`

   ![](./Assets/changingtourchto80.png)

- Both sites now listen on **port 80** since `reasonablecars.duckdns.org` uses port 80 by default, they differentiate by **host header**.



---
## 3️⃣ Configure Azure Application Gateway (AppGW)

### 🔧 App Gateway Setup Overview

| Component         | Value                             |
|------------------|-----------------------------------|
| Listener Type     | Multi-site                        |
| Listener Hostnames| `reasonablecars.duckdns.org`, `tourchboxz.duckdns.org` |
| Backend Pools     | Target: VM Private IP (10.0.0.4)  |
| Backend Port      | `80`                              |
| Host Header       | Overridden with matching DNS name |
| Probes            | Default or Custom (200–399)       |

### ✅ DNS Setup

- Used [DuckDNS](https://www.duckdns.org/) to register 2 domains:
  - `reasonablecars.duckdns.org`
  - `tourchboxz.duckdns.org`
- Updated DNS to point to **Application Gateway's Public IP**

---
### 4️⃣ Final Architecture Summary

- 🔒 NSG rules restrict RDP access and allow port 80 only.
- IIS hosts 2 websites on port 80 using host headers
- NSG is open on port 80; RDP is restricted by source IP
- 🔁 Azure Application Gateway routes based on host headers handles Layer 7 routing
- 🌐 Both websites accessible via:
  - `http://reasonablecars.duckdns.org`
  - `http://tourchboxz.duckdns.org`
  

---

## ✅ Final Access URLs
- 🌍 http://reasonablecars.duckdns.org

- 🌍 http://tourchboxz.duckdns.org

----

## 🔐 Phase 3: Enabling HTTPS with Let's Encrypt

### ✅ Tools & Methods

- **Tool:** Win-ACME for Let's Encrypt integration  
- **Cert Type:** Free SSL using HTTP-01 validation  
- **Install location:** `C:\SSL Certificate`

---

### 🧰 Steps to Generate Certificate (per site)

1. Run: `.\wacs.exe`
2. Choose **"Create certificate (full options)"**
3. Enter domain:
   - `reasonablecars.duckdns.org`  
   - or `tourchboxz.duckdns.org`
4. Choose validation method: **HTTP using `.well-known` folder**
   - Site 1 path: `C:\inetpub\wwwroot`
   - Site 2 path: `C:\inetpub\site2`
5. Save certificate as **PFX Archive** (e.g. `reasonablecars.pfx`)
6. Set password and save to vault
7. Skip Windows Cert Store (not exportable)

---

### 🔧 Configure App Gateway for HTTPS

| Listener Name                | Hostname                    | Port | Certificate         |
|-----------------------------|-----------------------------|------|---------------------|
| listener-https-reasonablecars | reasonablecars.duckdns.org | 443  | reasonablecars.pfx  |
| listener-https-tourchboxz     | tourchboxz.duckdns.org     | 443  | tourchboxz.pfx      |

- Edited routing rules to use new HTTPS listeners
- Verified websites load securely over port **443**

---

### 🔀 Redirect HTTP to HTTPS

- Used **Azure App Gateway** rule to redirect HTTP traffic to HTTPS
- No need to manually configure redirection inside IIS

---

### ✅ Final Secure Access URLs

- `https://reasonablecars.duckdns.org`
- `https://tourchboxz.duckdns.org`

---

### 🧠 Lessons Learned

- Use **PFX archive** for exportable certificates (not Windows Store)
- Upload certs manually to App Gateway
- Don't forget to configure **HTTP-to-HTTPS redirection**
- Let’s Encrypt certificates must be **renewed and re-uploaded every 60–90 days**

---

### 📌 Resources

- [Microsoft Docs – Windows IIS](https://learn.microsoft.com/en-us/iis/)
- [Azure NSG Overview](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
- [Azure ASG Overview](https://learn.microsoft.com/en-us/azure/virtual-network/application-security-groups)
- [Win-ACME](https://www.win-acme.com/)



## 🧠 Lessons Learned

- NSGs must be correctly configured for public traffic.
- App Gateway host header override is critical for IIS host-based binding.
- Default health probes must receive a 200 OK to pass.
- Test with:
  ```powershell
  curl -H "Host: tourchboxz.duckdns.org" http://localhost

---
## 📎 Resources

- [Microsoft Docs – Windows IIS](https://learn.microsoft.com/en-us/iis/)
- [Azure NSG Overview](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
- [Azure ASG Overview](https://learn.microsoft.com/en-us/azure/virtual-network/application-security-groups)

---

📝 For deployment challenges and troubleshooting notes, see [JOURNAL.md](./JOURNAL.md).
