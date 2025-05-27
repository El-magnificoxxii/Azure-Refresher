# Using Azure Firewall as a Gateway for All Outbound Traffic to the Internet


### Task 1: Creating the Spoke Virtual Networks
#### 1. Create Resource Group(Portal)

1. Go to Azure Portal
      - **Name:** `afw-test-ukw-rg` 
      - **Region:** UK West

#### 2. Build the First Spoke Virtual Network

1. Navigate to Virtual Networks → + Create

2. Basics:

    - **Name:** `spoke-01`
    - **Region:** UK West
    - **Address Space:** `10.13.1.0/24`

3. Subnets:

    - **default:** `10.13.1.0/26`
    - **services:** `10.13.1.64/26`


#### 3. Build the Second Spoke Virtual Network

Repeat Spoke‑01 steps, substituting:

1. Basics:

    - **Name:** `spoke-02`
    - **Region:** UK West
    - **Address Space:** `10.13.2.0/24`

2. Subnets:

    - **default:** `10.13.2.0/26`
    - **services:** `10.13.2.64/26`  

#### 4. Build the Third Spoke Virtual Network
Repeat Spoke‑01 steps, substituting:

1. Basics:

    - **Name:** `spoke-03`
    - **Region:** UK South
    - **Address Space:** `10.13.3.0/24`

2. Subnets:

    - **default:** `10.13.3.0/26`
    - **services:** `10.13.3.64/26`
  

### Task 2: Creating Hub Network Resources 

#### 1. Create a Firewall
1. Search for "Azure Firewall" → Click + Create
2. Basics:
      - **Name:** `lab-firewall`
      - **Firewall SKU:** Premium
      - **Firewall policy** Create new → `my-firewall-policy`
      - **Choose a virtual network** Create New → `hub-lab-net`
      - **Address space:** `10.12.0.0/16`
      - **IPv4 subnet:** `10.12.3.0/24`
      - **Public IP address:** `lab-firewall-ip`
  
3. Firewall Management NIC
      - **Enable Firewall Management NIC:** ✔️
      - **Subnet address space:** `10.12.5.0/26`
      - **Management public IP address:** Create New → `lab-firewall-ip-mgt`

4. Review + Create 

#### 2. Add Subnets to Hub Virtual Network (`hub-lab-net`)
1. Add the Default Subnet:
      - **Default** (`Default` purpose): `10.12.1.0/24`  
      - **AzureBastionSubnet** (`Azure Bastion` purpose): `10.12.2.0/24`  
      - **GatewaySubnet** (`Virtual network gateway` purpose): `10.12.4.0/24`  


#### 3. Deploy Azure Bastion

1. Search **Azure Bastion** → **+ Create**  
2. **Settings**:  
      - **Name:** `lab-bastion`  
      - **Tier:** Standard  
      - **Virtual network:** `hub-lab-net` (AzureBastionSubnet)  
      - **Public IP:** `lab-bastion-ip`  
3. Review + Create


#### 4. Deploy Virtual Network Gateway

1. Search **Virtual network gateways** → **+ Create**
2. **Basics**:  
      - **Name:** `lab-gateway`  
      - **SKU:** `VpnGw2` (Generation 2)  
      - **Virtual network:** `hub-lab-net` (GatewaySubnet)  
      - **Gateway type:** VPN  
      - **VPN type:** Route-based  
      - **Active-active mode:** Disabled  
      - **BGP:** Disabled  
      - **Public IP:** `lab-gateway-ip`  

3. Review + Create
   
### Task 3: Configure Virtual Network Peerings

#### 1. Create a Virtual Network Peering between the First Spoke Virtual Network and the Hub Virtual Network

1. On the Spoke-01 virtual network > **Peerings** > **+ Add**.
2. Remote Virtual Network Summary 

    - **Name:** hublabnettospoke01
    - **Virtual Network:** hub-lab-net

    - **Allow 'hub-lab-net' to access 'spoke-01 traffic':** ✔️
   
    - **Allow 'hub-lab-net' to receive forwarded traffic from 'spoke-01':** ✔️


3. Local Virtual Network Summary
    - **Name:** spoke01tohublabnet
    - **Virtual Network:** spoke-01
    - **Allow 'spoke-01' to access 'hub-lab-net traffic':** ✔️
    - **Allow 'spoke-01' to receive forwarded traffic from 'hub-lab-net traffic':** ✔️

#### 2. Create a Virtual Network Peering between the Second Spoke Virtual Network and the Hub Virtual Network

1. On the Spoke-02 virtual network > **Peerings** > **+ Add**.
2. Remote Virtual Network Summary 

    - **Name:** hublabnettospoke02
    - **Virtual Network:** hub-lab-net
    - **Allow 'hub-lab-net' to access 'spoke-02 traffic':** ✔️
    - **Allow 'hub-lab-net' to receive forwarded traffic from 'spoke-02':** ✔️


3. Local Virtual Network Summary
    - **Name:** spoke02tohublabnet
    - **Virtual Network:** spoke-02
    - **Allow 'spoke-02' to access 'hub-lab-net traffic':** ✔️
    - **Allow 'spoke-02' to receive forwarded traffic from 'hub-lab-net traffic':** ✔️
      

#### 3. Create a Virtual Network Peering between the Third Spoke Virtual Network and the Hub Virtual Network

1. On the Spoke-03 virtual network > **Peerings** > **+ Add**.
2. Remote Virtual Network Summary 

    - **Name:** hublabnettospoke03
    - **Virtual Network:** hub-lab-net
    - **Allow 'hub-lab-net' to access 'spoke-03 traffic':** ✔️
    - **Allow 'hub-lab-net' to receive forwarded traffic from 'spoke-03':** ✔️


3. Local Virtual Network Summary
    - **Name:** spoke03tohublabnet
    - **Virtual Network:** spoke-03
    - **Allow 'spoke-03' to access 'hub-lab-net traffic':** ✔️
    - **Allow 'spoke-03' to receive forwarded traffic from 'hub-lab-net traffic':** ✔️
      

### Task 4: Create Route Tables for Forced Tunneling

#### Create a Route Table to Route traffic from Spoke-01 and Spoke-02 to Azure Firewall
1. Navigate to **Route Tables** → Click **+ Create**
2. Instance Details:
   
      - **Name:** spokes-ukw-to-hub-routes
      -  **Region:** UK West
  
3. Select 'spokes-ukw-to-hub-routes' > Settings > Routes > + Add
4. Include the following informationn
      - **Name:** to-firewall
      - **Destination type:** IP Address
      - **IP Address:** 0.0.0.0/0
      - **Next hop type:** virtual appliance
      - **next hop address:** 10.12.3.4
        
5. Navigate to 'spokes-ukw-to-hub-routes' > subnets > associate the following subnet


| Subnet Name  | Virtual Network |
| ------------- | ------------- |
| default  | `spoke-01`  |
| default  | `spoke-02`  |
| services  | `spoke-01`  |
| services  | `spoke-02`  |
  
#### Create another Route Table to Route traffic from Spoke-03 to Azure Firewall

6. Repeat the same steps as spokes-ukw-to-hub-routes:
7. Instance Details:
      -**Region:** Uk south
      -**Name:** spokes-uks-to-hub-routes

8. Navigate to 'spokes-uks-to-hub-routes' > Settings > Routes > + Add
9. Include the following informationn
      - **Name:** to-firewall
      - **Destination type:** IP Address
      - **IP Address:** 0.0.0.0/0
      - **Next hop type:** virtual appliance
      - **next hop address:** 10.12.3.4
  
5. Go to 'spokes-uks-to-hub-routes' > subnets > associate the following subnet


| Subnet Name  | Virtual Network |
| ------------- | ------------- |
| default  | `spoke-03` |
| services  | `spoke-03`  |


### Task 5: Configure Rules in the Created Firewall Policies 

1. Open **Firewall policies** → **my-firewall-policy** → **Rules*

      - **Name:** rfc1918-collection
      - **rule collection type:** Network
      - **priority:** 1000
      - **rule collection action:** deny
      - **rule name:** block-intranet-traffic
      - **source type:** ip address
      - **source:** 10.13.1.0/24,10.13.2.0/24,10.13.3.0/24
      - **protocol:** TCP + UDP
      - **destination ports:** *
      - **destination type:** ip address
      - **destination:** 10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
        
2. Repeat the same steps and add another Rule Collectiom

      - **Name:** internet-collection
      - **rule collection type:** Network
      - **priority:** 10000
      - **rule collection action:** allow
      - **rule name:** to-internet-rule
      - **source type:** ip address
      - **source:** 10.13.1.0/24,10.13.2.0/24,10.13.3.0/24
      - **protocol:** TCP + UDP
      - **destination ports:** *
      - **destination type:** ip address
      - **destination:** *

### Task 6: Deploy Test Virtual Machines


| Virtual Machine Name  | Subnet | Virtual Network |
| ------------- | ------------- | ------------- |
| hub-vm-01 | DefaultSubnet | hub-lab-net |
| spoke-01-vm | default | spoke-01 | 
| spoke-02-vm | default | spoke-02 | 
| spoke-03-vm | default | spoke-03 | 


### Task 7: Configure a Point to Site VPN
#### 1. Generate self signed certificate for Point to Site Configuration
1. On your local computer, run 'Windows Powershell ISE' as admin
2. Run the script below to generate Root certificate

 ```powershell

$params = @{
    Type = 'Custom'
    Subject = 'CN=P2SRootCert'
    KeySpec = 'Signature'
    KeyExportPolicy = 'Exportable'
    KeyUsage = 'CertSign'
    KeyUsageProperty = 'Sign'
    KeyLength = 2048
    HashAlgorithm = 'sha256'
    NotAfter = (Get-Date).AddMonths(24)
    CertStoreLocation = 'Cert:\CurrentUser\My'
}
$cert = New-SelfSignedCertificate @params



```


3. On the same text extension, run the script below to generate a Child certificate

 ```powershell

$params = @{
    Type = 'Custom'
    Subject = 'CN=P2SChildCert'
    DnsName = 'P2SChildCert1'
    KeySpec = 'Signature'
    KeyExportPolicy = 'Exportable'
    KeyLength = 2048
    HashAlgorithm = 'sha256'
    NotAfter = (Get-Date).AddMonths(18)
    CertStoreLocation = 'Cert:\CurrentUser\My'
    Signer = $cert
    TextExtension = @(
     '2.5.29.37={text}1.3.6.1.5.5.7.3.2')
}
New-SelfSignedCertificate @params



```

#### 2. Export the Root Certificate

4. Select 'manage user certificate' > Personal > Certicates > Export P2SRootCert 
5. From the Certificate Export Wizard, select the format 'Base-64 encoded X.509'
6. Select the path the file will be downloaded to on your local computer
7. Select Next and Finish

#### 3. Export the Child Certificate

8. Select 'manage user certificate' > Personal > Certicates > Export P2SChildCert
9. From the Certificate Export Wizard, select the format 'Personal Information Exchange -PKCS #12'
      - **Include all the certificates in the certification path if possible** ✔️
      - **Enable certificate privacy** ✔️

10. Include the password and encryption
11. Select the path the file will be downloaded to on your local computer
12. Select Next and Finish

#### 4. Add the Root Certificate to your Virtual Network Gateway 

13. Go to Azure Portal
14. Open Virtual Network Gateway > lab Gateway > Point to site configuration > Configure now
15. Copy the key from the downloaded root certificate into the Point to site configuration and save
16. Select Download VPN Client on your local computer

#### 5. Add the Child Certificate to your Local Computer
17. Download and Install Azure VPN Client from Microsoft Stores
18. From the Azure VPN Client interface, Select +
19. Import the Downloaded VPN Client
20. Include the following information:
      - **Authentication Type:** Certificate
      - **Client Information:** P2SChildCert

21. Ensure the VPN Client is showing connected

_Connection has been established between your local computer and your hub-lab-net network_


### Task 8: Testing Network traffic to your Firewall from Spoke Networks

#### 1. Create Firewall Log Workspace to save Firewall Logs
1.  Search for **Log analytics Workspace** → Click + Create `firewalldiagnosticworkspace`
2.  Save the settings

#### 2. Connect to the VM in the Spoke Network
3. Connect to `spoke-01` vm via Azure Bastion
4. Open powershell as administrator
5. Run the script below


_Once the script is running, network traffic is imitated on your network_

#### 3. Monitor the traffic in the Firewall 
6. In your Azure portal, select **Firewall** → Click `lab-firewall`
7. Select **Logs** → `Network rule log data`

_You should start seeing netowork log information from spoke01-vm and lab firewall_


  

        









