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

#### 2. Add Subnets in Hub Virtual Network 
1. Search for "Virtual Networks" → Click 'hub-lab-net'
2. Select Subnet →  Click '+ Subnet'
3. Add the Default Subnet:
      - **Subnet purpose:** Default
      - **Name:** Default


   


   
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







