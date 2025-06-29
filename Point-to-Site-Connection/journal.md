# Lessons from the Field: Challenges Deploying Azure Firewall in a Hub-and-Spoke Architecture

## Introduction
Deploying network security in the cloud can be tricky, especially when dealing with complex architectures and platform limitations. In this article, I share my hands-on experience deploying an Azure Firewall within a hub-and-spoke network topology, the challenges I faced, and how I eventually overcame them.

## 1. Understanding the Cloud Architecture
The deployment was based on a hub-and-spoke architecture:
- Three spoke VNets were connected to a central hub VNet.
- The hub hosted shared services including Azure Firewall, VPN Gateway, and Bastion.
- Each spoke VNet contained isolated workloads with no direct internet access. The hub network served as a central point of connection for all spoke networks.

## 2. Routing Traffic via Azure Firewall
My goal was to route all outgoing traffic from the three spoke networks through the Azure Firewall deployed in the hub. The architecture was expected to allow internet-bound traffic while blocking access to specific external IP ranges.

With the firewall policies in place:
- The first rule (`rfc1918-collection`) prevents VMs in the spoke networks from communicating with private IP ranges (RFC1918 addresses).
- The second rule (`internet-collection`) allows VMs to access the internet.

This ensures each network remains isolated and secure, preventing malicious access while still enabling VMs to reach the internet for updates and other necessary operations.

## 3. Using ARM Template to Deploy Resources
After realizing that I couldn’t create the `/24` subnet for the firewall directly in the portal, I had to delete some existing resources on the virtual network. To avoid starting from scratch, I downloaded the ARM template of the deployed resources for redeployment.

During redeployment, I encountered several issues:
- **Incremental Deployment Mode:** ARM templates use incremental mode by default, which doesn't delete resources not listed in the template. However, for subnets, ARM treats the list as the full set—even in incremental mode—and will remove any subnet not specified.
- **Circular Dependency:** There were dependencies where the hub relied on the spokes and vice versa. This caused deployment errors because Azure couldn’t determine which resource to deploy first.

**Solution:** I removed the mutual `dependsOn` references so that hub and spoke deployments were independent of each other.

## 4. Role of Firewall, Bastion, and Virtual Network Gateway
Each core component in the hub had a specific role:
- **Azure Firewall:** Acts as a centralized control point for filtering traffic from the spoke networks based on configured policies.
- **Azure Bastion:** Provides secure RDP/SSH access to VMs in the spoke networks without needing public IPs.
- **Virtual Network Gateway:** Enables secure connections from on-premises devices (like a developer’s laptop) to Azure, making them part of the same virtual network.

## 5. Firewall Subnet Masking Issue
When manually deploying Azure Firewall after creating the subnets, I encountered an error with the `/24` subnet. For forced tunneling scenarios, Azure requires a `/26` subnet or smaller. I resolved this by creating the Azure Firewall during the virtual network creation process, ensuring proper subnet configuration.

## 6. VPN Gateway Certificate Setup Issue
To configure Point-to-Site VPN using certificates, I followed Microsoft’s guide:  
https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-certificates-point-to-site

Initially, I ran the certificate generation scripts in separate PowerShell sessions, which caused issues. Running the entire script in a single session resolved the problem and ensured proper linkage of root and client certificates.

## 7. Python Script Failures and ChatGPT Help
To simulate traffic and test the firewall, I used a Python script. Initially, it failed due to syntax errors and authentication problems. ChatGPT helped me debug and improve the script. The updated version successfully generated outbound traffic through the firewall.

## 8. Is the Firewall Working? Verifying via Logs
Even after generating traffic, I wasn’t certain the firewall was working correctly. I used Azure Monitor and Log Analytics to confirm traffic flows and validate rule hits, which confirmed the firewall's functionality.

## 9. Resource Group Misalignment Across Regions

In my hub-and-spoke setup, I mistakenly placed all resources—including the hub (UK West) and one spoke in a different region (UK South)—into a single resource group. While it worked functionally, this setup goes against Microsoft’s best practices.

> "Place resources in the same region as the resource group. If resources span regions, use separate resource groups."  
> — [Microsoft Docs](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-setup-guide/organize-resources)

### Why It Matters
Keeping region-specific resources in separate resource groups helps:
- Improve regional fault isolation
- Align with Azure governance standards
- Simplify resource management and scaling

### Corrected Approach

| Region   | Recommended Resource Group     | Example Resources                      |
|----------|--------------------------------|----------------------------------------|
| UK West  | `rg-network-ukwest`            | Hub VNet, UK West spokes, Bastion, VPN |
| UK South | `rg-network-uksouth`           | UK South spoke VNet                    |

Following this guidance ensures your architecture is more compliant, maintainable, and ready for production-scale operations.


## 10. Reusing the ARM Template: Circular Dependency Challenge
After successfully deploying the hub-and-spoke network architecture via the Azure Portal, I downloaded the exported ARM template with the intention of archiving it and reusing it for automation in future environments.

However, when I tried redeploying this exported template, I encountered the following error:

<pre> ``` Deployment template validation failed: 'Circular dependency detected on resource: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/test/providers/Microsoft.Network/virtualNetworks/hub-lab-net'. Please see https://aka.ms/arm-syntax-resources for usage details.' (Code: InvalidTemplate) ``` </pre>



This was confusing at first because I had already deployed the same configuration manually via the GUI. After a careful review, I discovered that several `dependsOn` declarations across the hub and spoke resources created a **circular dependency chain**. The exported template was accurate in terms of Azure's deployment representation, but **not optimized for redeployment as-is**.

### 🔍 Diagnosis

I reviewed all `dependsOn` entries and discovered that many subnets under `hub-lab-net` had `dependsOn` pointing to the parent virtual network — which itself was being referenced elsewhere. This recursive loop was not tolerated by the ARM engine.

To isolate the issue, I:

- Identified all resources using `parameters('virtualNetworks_hub_lab_net_name')` with a `dependsOn`.
- Attempted to comment out `dependsOn` entries — but discovered that **ARM templates do not support comments** like `//` or `/* */`, making the template invalid JSON.
- Used [JSONLint](https://jsonlint.com/) to validate the structure and avoid syntax-related deployment errors.
- Gradually removed non-essential `dependsOn` lines to break the circular logic.

### ⚙️ Solution

Eventually, I removed or disabled the `dependsOn` directives from subnets and child resources that were already implicitly tied to the virtual network through their hierarchy. Once those dependencies were removed, **the template deployed successfully**.

I maintained two versions of the template for documentation:

- [`template.json`](https://github.com/El-magnificoxxii/Azure-Refresher/blob/main/Point-to-Site-Connection/templates/broken/template.json): Captures the original exported ARM with circular dependency.
- [`new_template.json`](https://github.com/El-magnificoxxii/Azure-Refresher/blob/main/Point-to-Site-Connection/templates/working/new_template.json): The working version after pruning `dependsOn` statements and validating the structure.

---

## ✅ Conclusion: Key Takeaways

- Always check subnet requirements when deploying services like Azure Firewall, Bastion, and VPN Gateway.
- Be mindful of Azure's regional limitations and how they affect your resource planning.
- Run certificate scripts in the same PowerShell session to avoid broken VPN setups.
- Use logs (not assumptions) to verify firewall behavior.
- **Refactor exported templates** before reusing them to avoid circular dependencies.
- **Don’t comment in ARM templates** — validate with tools like JSONLint instead.
- Use tools like Firewall Manager and AI tools (like ChatGPT) when troubleshooting.

This experience reinforced the importance of meticulous planning, deep template analysis, and validation in cloud network deployments.







## Conclusion: Key Takeaways
- Always check subnet requirements when deploying services like Azure Firewall, Bastion, and VPN Gateway.
- Be mindful of Azure's regional limitations and how they affect your resource planning.
- Run certificate scripts in the same PowerShell session to avoid broken VPN setups.
- Use logs (not assumptions) to verify firewall behavior.
- Don’t hesitate to leverage tools like Firewall Manager and AI tools when troubleshooting.

This experience reinforced the importance of meticulous planning and validation in cloud network deployments. 

