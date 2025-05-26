# Using Azure Firewall as a Gateway for All Outbound Traffic to the Internet


### Task 1: Creating the  Spoke Networks
#### 1. Create Resource Group (GUI)
1. Go to Azure Portal

2. Search for "Resource Groups" → Click + Create

3. Basics Tab:

      - **Name:** afw-test-ukw-rg

      - **Region:** UK West
        (image)

#### 2. Build the First Spoke Virtual Network
1. Search for "Virtual Networks" → + Create

2. Basics Tab:

    - **Name:** spoke-01

    - **Region:** UK West

    - **Address Space:** 10.13.1.0/24

3. Subnets Tab:

    - **Add default subnet:** 10.13.1.0/26

    - **Add services subnet:** 10.13.1.64/26
Spoke VNet Setup (image)

#### 3. Build the Second Spoke Virtual Network
Repeat the same steps as the first spoke virtual network

1. Basics Tab:

    - **Name:** spoke-02

    - **Region:** UK West

    - **Address Space:** 10.13.2.0/24

2. Subnets Tab:

    - **Add default subnet:** 10.13.2.0/26

    - **Add services subnet:** 10.13.2.64/26
   Spoke2 VNet Setup (image)

#### 4. Build the Third Spoke Virtual Network
Repeat the same steps as the first spoke virtual network

1. Basics Tab:

    - **Name:** spoke-03

    - **Region:** UK South

    - **Address Space:** 10.13.3.0/24

2. Subnets Tab:

    - **Add default subnet:** 10.13.3.0/26

    - **Add services subnet:** 10.13.3.64/26
   Spoke3 VNet Setup (image)

### Task 2: Creating Resources in Hub Virtual Network 

#### 1. Create a Firewall
1. Search for "Azure Firewall" → Click + Create
2.  Instance detail Summary
      - **Name:** lab-firewall
      - **Firewall SKU:** Premium
      - **Firewall policy** Add new → 'my-firewall-policy'
      - **Choose a virtual network** Create New → 'hub-lab-net'
      - **Address space:** 10.12.0.0/16
      - **IPv4 subnet:** 10.12.3.0/24
      - **Public IP address:** lab-firewall-ip
  
3. Firewall Management NIC
      - **Enable Firewall Management NIC:** ✔️
      - **Subnet address space:** 10.12.5.0/26
      - **Management public IP address:** Create New → 'lab-firewall-ip-mgt'

4. Review + Create → Create

#### 2. Add Other Subnets in Hub Virtual Network 
1. Search for "Virtual Networks" → Click 'hub-lab-net'
2. Select Subnet →  Click '+ Subnet'
3. Add the Default Subnet:
      - **Subnet purpose:** Default
      - **Name:** Default
      - **IPv4 address range:** 10.12.1.0/24
4. Add the Azure Bastion Subnet:
      - **Subnet purpose:** Azure Bastion
      - **Name:** AzureBastionSubnet
      - **IPv4 address range:** 10.12.2.0/24
5. Add the Virtual Network Gateway Subnet:
      - **Subnet purpose:** Virtual Network Gateway
      - **Name:** GatewaySubnet
      - **IPv4 address range:** 10.12.4.0/24


#### 3. Create Azure Bastion

1. Search for "Azure Bastion" → Click + Create
2.  Instance detail Summary
      - **Name:** lab-firewall
      - **Tier** standard
      - **Public IP address name:** lab-bastion-ip
      - **Public IP address name:** AzureBastionSubnet (10.12.2.0/24)
        

#### 4. Create a Virtual Network Gateway

1. Search for "virtual Network Gateway" → Click + Create
2. Instance detail Summary
      - **Name:** lab-gateway
      - **SKU:** VpnGw2
      - **Generation:** Generation2
      - **Subnet:** GatewaySubnet (10.12.4.0/24)
      - **Gateway type:** Vpn
      - **VPN type:** RouteBased
      - **Enable active-active mode:** Disabled
      - **Configure BGP:** Disabled
      - **Public IP address:** lab-gateway-ip






        


   


   
### Task 3: Creating Virtual Network Peering between the Networks

#### 1. Create a Virtual Network Peering between the First Spoke Virtual Network and the Hub Virtual Network

1. Open Spoke-01 virtual network > Peerings > + Add.
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

1. Open Spoke-02 virtual network > Peerings > + Add.
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

1. Open Spoke-03 virtual network > Peerings > + Add.
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
      

### Task 4: Create a Routing Table to route Network Traffic from the Spoke Networks to Azure Firewall in Hub Network

#### Create a Route Table to Route traffic from Spoke-01 and Spoke-02 to Azure Firewall
1. Search for "Route Tables" → Click '+ Create'
2. Instance Details:
   
      - **Name:** spokes-ukw-to-hub-routes
      - **Virtual Network:** spoke-03
      -  **Region:** UK West
  
3. Select 'spokes-ukw-to-hub-routes' > Settings > Routes > + Add
4. Include the following informationn
      - **Name:** to-firewall
      - **Destination type:** IP Address
      - **IP Address:** 0.0.0.0/0
      - **Next hop type:** virtual appliance
      - **next hop address:** 10.12.3.4
        
5. Go to 'spokes-ukw-to-hub-routes' > subnets > associate the following subnet


| Subnet Name  | Virtual Network |
| ------------- | ------------- |
| default  | spoke-01  |
| default  | spoke-02  |
| services  | spoke-01  |
| services  | spoke-02  |
  
#### Create another Route Table to Route traffic from Spoke-03 to Azure Firewall

6. Repeat the same steps as spokes-ukw-to-hub-routes:
7. Instance Details:
      -**Region:** uk south
      -**Name:** spokes-uks-to-hub-routes

8. Go to 'spokes-uks-to-hub-routes' > Settings > Routes > + Add
9. Include the following informationn
      - **Name:** to-firewall
      - **Destination type:** IP Address
      - **IP Address:** 0.0.0.0/0
      - **Next hop type:** virtual appliance
      - **next hop address:** 10.12.3.4
  
5. Go to 'spokes-uks-to-hub-routes' > subnets > associate the following subnet


| Subnet Name  | Virtual Network |
| ------------- | ------------- |
| default  | spoke-03 |
| services  | spoke-03  |


### Task 5: Configure Rules in the Created Firewall Policies 

1. Search for "firewall Policies" → Click 'my-firewall-policy'
2. select Rules > Add a Rule Collection
3. Add the following information

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
        
4. Repeat the same steps and add another Rule Collectiom

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

### Task 6: Create Virtual Machines in their respective Subnets



| Virtual Machine Name  | Subnet | Virtual Network |
| ------------- | ------------- | ------------- |
| hub-vm-01 | DefaultSubnet | hub-lab-net |
| spoke-01-vm | default | spoke-01 | 
| spoke-02-vm | default | spoke-02 | 
| spoke-03-vm | default | spoke-03 | 


### Task 7: Configure a Point to Site Connection
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

3. Select 'manage user certificate' > Personal > Certicates > Export P2SRootCert 
4. From the Certificate Export Wizard, select the format 'Base-64 encoded X.509'
5. Select the path the file will be downloaded to on your local computer
6. Select Next and Finish

#### 3. Export the Child Certificate

7. Select 'manage user certificate' > Personal > Certicates > Export P2SChildCert
8. From the Certificate Export Wizard, select the format 'Personal Information Exchange -PKCS #12'
      - **Include all the certificates in the certification path if possible** ✔️
      - **Enable certificate privacy** ✔️

9. Include the password and encryption
10. Select the path the file will be downloaded to on your local computer
11. Select Next and Finish

#### 4. Add the Root Certificate to ypur Virtual Network Gateway 

1. Go to Azure Portal
2. Open Virtual Network Gateway > lab Gateway > Point to site configuration > Configure now
3.  
4.





  

        









