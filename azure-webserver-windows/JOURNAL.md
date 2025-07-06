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

- Installed IIS on the VM using:

  ```powershell
  Install-WindowsFeature -name Web-Server -IncludeManagementTools
  ```

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

## 🧪 Local Testing Inside the VM

- Accessed both sites from within the VM:
  - `http://localhost` → ✅ Website 1 loaded
  - `http://localhost:8080` → ✅ Website 2 loaded
- Confirmed both sites were active and IIS was properly configured.

---

## 🔍 Confirming Port Binding

Ran the following in PowerShell to verify the server was listening:

```powershell
netstat -ano | findstr :8080
```

Output showed:

```
TCP    0.0.0.0:8080      0.0.0.0:0     LISTENING     4  
TCP    [::]:8080         [::]:0        LISTENING     4
```

✅ Confirmed IIS was listening on port 8080 for both IPv4 and IPv6.

---

## ❌ External Access to Port 8080 Failed

- Accessing `http://<public-ip>:8080` from the local browser failed.
- Ran:

  ```powershell
  Test-NetConnection -ComputerName <public-ip> -Port 8080
  ```

  Got:

  ```
  TcpTestSucceeded : False
  ```

Indicated a network accessibility issue despite local success.

---

## 🧠 Troubleshooting Summary

| Area Checked          | Result                      |
|-----------------------|-----------------------------|
| IIS configuration     | ✅ Correct via localhost     |
| Port listening        | ✅ Confirmed with `netstat`  |
| Windows Firewall      | 🔄 Disabled (not the issue)  |
| Azure NSG             | 🛑 Suspected issue           |

---

## 🧩 Root Cause Found in NSG

The NSG had an inbound rule like:

| Field        | Value               |
|--------------|---------------------|
| Source       | Any                 |
| Destination  | VM Private IP `/32` ❌ |
| Port         | 8080                |
| Protocol     | TCP                 |
| Action       | Allow               |

### ❗ Problem:

The NSG is attached to the NIC (public IP mapped to it). Restricting destination to private IP caused the rule to not match incoming traffic.

---

## 🛠️ Fix – Correct NSG Rule

Replaced the rule with:

| Field        | Value   |
|--------------|---------|
| Source       | Any     |
| Destination  | Any ✅   |
| Port         | 8080    |
| Protocol     | TCP     |
| Action       | Allow   |
| Priority     | 200     |

✅ NSG now allows public traffic to port 8080.

---

## ✅ Final Confirmation

- Re-ran:

  ```powershell
  Test-NetConnection <public-ip> -Port 8080
  ```

  Result:

  ```
  TcpTestSucceeded : True
  ```

- Opened `http://<public-ip>:8080` in browser — **Second website loaded successfully** 🎉

---

## 📌 Challenges Faced and How They Were Resolved

| Challenge                         | Solution                                |
|----------------------------------|------------------------------------------|
| Website not loading externally   | Verified local IIS config using localhost |
| Unsure if port was listening     | Used `netstat -ano`                      |
| Suspected Windows Firewall       | Confirmed it was off                     |
| NSG misconfiguration             | Fixed Destination = Any                  |
| No clear Azure error             | Used `Test-NetConnection` to isolate     |

---

## 🧠 Lessons Learned

- ✅ Start troubleshooting from the inside — local testing is powerful.
- ✅ NSG destination filtering can unintentionally block traffic.
- ✅ Use `netstat` to confirm port bindings.
- ✅ Use `Test-NetConnection` for remote port testing.
- ✅ Narrowing NSG rules too much may block expected traffic.

---

## 🚀 Next Steps

- Add host headers for domain-based routing instead of ports.
- Use HTTPS with Let's Encrypt or a custom certificate.
- Deploy Azure Front Door or Application Gateway for reverse proxy.
- Re-enable Windows Firewall and explicitly allow only required ports.

---

