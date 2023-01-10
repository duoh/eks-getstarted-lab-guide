## Set up Environment for the workshop
Initially, a workshop environment comprises a default VPC with Internet Gateway and 3 public subnets.
In this part we are going to create 3 private subnets in order to allow EKS cluster to be placed on these subnets.
Nat Gateway also is used for the Internet outbound traffic. Last, a newly-created route table is associated with 3 private subnets.  

## Architecture Diagram

![pics](/pics/env-setup.png)  


## Preparing infra/network environment
* Go to VPC console (https://console.aws.amazon.com/vpc)

* Create 3 new subnets in a default vpc (IPv4 CIDR 172.31.0.0/16) with the following parameters
    * Subnet name: private-subnet-a, Availability Zone: ap-southeast-1a, IPv4 CIDR block: 172.31.48.0/20
    * Subnet name: private-subnet-b, Availability Zone: ap-southeast-1b, IPv4 CIDR block: 172.31.64.0/20
    * Subnet name: private-subnet-c, Availability Zone: ap-southeast-1c, IPv4 CIDR block: 172.31.80.0/20

![pics](/pics/subnets.png)  
  
* Create a Nat Gateway and place it on one of public subnets
    * Select one of subnets whose name is empty
    * Connectivity type: Public
    * Get a public IP address by clicking on 'Allocate Elastic IP'

![pics](/pics/natgw.png)  
  
* Create a route table for private subnets and add route entries
    * Name: private-route-table
    * Choose a default VPC
    * Click on 'Create route table'
    * Tick on checkbox in front of the route table we have just created
    * On Subnet associations tab, click on 'Edit subnet associations'
    * Select 3 private subnets (name beginning with private-*)
    * On Routes tab, click on 'Edit routes'
    * Click on 'Add route' and assign with the following
        * Destination: 0.0.0.0/0
        * Target: Choose Nat Gateway (it will populate NatGW-id that we created in previous section)

![pics](/pics/routes.png)  
