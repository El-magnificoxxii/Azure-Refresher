# Windows VM Web Server Deployment on Azure (with Two Websites)

This project demonstrates the deployment of a Windows Virtual Machine on Azure configured to run two web servers on different ports using IIS. It includes setup of network security via NSG and ASG and basic web hosting configuration.

---

## 🌐 1. Create Virtual Network (VNet)

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

## 🔐 2. Create Application Security Groups (ASG)

| Name        | Region   | Resource Group     |
|-------------|----------|--------------------|
| win-vm-asg  | UK West  | webapp-test-rg     |
| lin-vm-asg  | UK West  | webapp-test-rg     |

---

## 💻 3. Create Windows Virtual Machine

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

## 🛠️ 4. Configure Web Server (IIS)

### A. Connect to the VM

1. Download the RDP file from the Azure portal.
2. Log in using the credentials set during deployment.

### B. Install IIS via PowerShell

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

### C. Confirm IIS Is Running

1. Open a browser on your local machine.
2. Navigate to your VM’s **public IP**.
3. You should see the default IIS landing page.

### D. Deploy Your First Website

1. Go to `C:\inetpub\wwwroot`
2. Remove the default files (including `iisstart.htm`)
3. Paste your own website files into the folder

---

## 🌐 5. Host a Second Website on the Same VM (Different Port)

### A. Create Folder for Site 2

1. Navigate to `C:\inetpub\`
2. Create a new folder: `site2`
3. Paste the second website’s files into `site2`

### B. Configure Second Website in IIS

1. Open **IIS Manager**
2. In the left panel, right-click **Sites** > **Add Website**
   - **Site Name**: `Site2`
   - **Physical Path**: `C:\inetpub\site2`
   - **Binding**:
     - **IP Address**: All Unassigned
     - **Port**: `8080`
     - **Host name**: *(leave blank)*
3. Click **OK** to create the site

---

## 🔒 6. Update NSG to Allow Port 8080

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

---

## 📌 Summary

- VM hosts two websites on ports `80` and `8080`
- IIS used for configuration
- NSG and ASG applied for network control
- Sites accessible publicly with correct ports

---

## 📎 Resources

- [Microsoft Docs – Windows IIS](https://learn.microsoft.com/en-us/iis/)
- [Azure NSG Overview](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
- [Azure ASG Overview](https://learn.microsoft.com/en-us/azure/virtual-network/application-security-groups)

---

📝 For deployment challenges and troubleshooting notes, see [JOURNAL.md](./JOURNAL.md).
