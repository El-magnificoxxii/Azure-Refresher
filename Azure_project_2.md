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


### Task 2: Creating the Hub Network

1. Basics Tab:

    - **Name:** hub-lab-net

    - **Region:** UK West

    - **Address Space:** 10.12.0.0/16

2. Subnets Tab:

    - **Add default subnet:** 10.12.1.0/24

  
   Spoke3 VNet Setup (image)

   
### Task 3: Creating Virtual Network Peering between the First Spoke Network and Hub Network

1. Open Spoke-01 virtual network > Peerings > + Add.
2. Remote Virtual Network Summary 

    - **Name:** hublabnettospoke01
    - **Virtual Network:** hub-lab-net

    - **Allow 'hub-lab-net' to access 'spoke-01 traffic':** ✔️



