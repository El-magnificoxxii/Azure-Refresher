# JOURNAL.md

## 🧾 Deployment Journal – Windows VM Web Server on Azure

This journal documents the real-world experience, challenges, and observations during the deployment of a Windows VM in Azure, used to host two IIS-based websites on different ports.

---

## ✅ Initial Setup Steps

- Created a Virtual Network `webapp-test-vnet` with address space `10.0.0.0/22` and subnet `10.0.0.0/24`.  
- Created two Application Security Groups (ASG):  
  - `win-vm-asg` for the Windows VM  
  - `lin-vm-asg` for a potential Linux VM  
- Deployed a Windows Server 2022 VM named `win-vm` with:  
  - Public IP  
  - Inbound ports: RDP, HTTP, HTTPS  
  - Subnet: `default`

---


## 🌐 Web Server Configuration

- Confirmed IIS was installed by visiting the VM's public IP in a browser.
- Navigated to `C:\inetpub\wwwroot`, deleted default `iisstart.htm`, and replaced with a custom site.
- Confirmed website served successfully on port 80.

---

## 🔄 NSG and ASG Testing

- Attached `win-vm-asg` to the VM under **Networking > Application Security Groups**.
- After applying ASG, lost access to the web server.

### 🛑 Issue Faced

- NSG rule incorrectly used the **public IP** of the VM as the destination.
- This blocked traffic even though port 80 was open.

### 🛠️ Resolution

- Updated the NSG rule:
  - Changed **Destination** to use the **private IP address** or **CIDR** of the VM.
  - Access was restored.

✅ Website loading restored after adjusting NSG rule.

---

## 🌐 Hosting a Second Website on the Same VM (Port 8080)

### A. Create Second Website Folder

- Logged into the VM via RDP.
- Navigated to `C:\inetpub\`.
- Created a folder named `site2`.
- Copied the second website’s files into `site2`.

### B. Add Second Website in IIS

- Opened **IIS Manager**.
- Right-clicked **Sites** > **Add Website**.
- Provided the following values:
  - **Site Name**: `Site2`
  - **Physical Path**: `C:\inetpub\site2`
  - **Binding**:
    - **IP Address**: All Unassigned
    - **Port**: `8080`
    - **Host name**: *(left blank)*
- Clicked **OK** to create the second website.

---

## 🔒 Update NSG to Allow Port 8080

- Navigated to NSG associated with the VM.
- Created a new inbound rule to allow traffic on port `8080`.

| Field         | Value   |
|---------------|---------|
| **Source**    | Any     |
| **Destination** | Any   |
| **Port**      | 8080    |
| **Protocol**  | TCP     |
| **Action**    | Allow   |
| **Priority**  | 200     |

✅ Successfully accessed the second website via:

