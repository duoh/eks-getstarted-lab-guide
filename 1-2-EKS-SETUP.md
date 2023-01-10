## Preparing EKS cluster
Before creating an EKS cluster, we need to create a cluster service role to allow the control plane cluster to manage AWS resources on our behalf, 
by creating a role with with a managed policy, AmazonEKSClusterPolicy to allow EKS service assume 
* Go to Roles on IAM console (https://console.aws.amazon.com/iamv2/home#/roles)
    * Click on 'Create role'  and choose options as follows
    * Trusted entity type: AWS service
    * Use case: EKS - Cluster
    ![pics](/pics/eks-usecase.png)  
    * Role name: myAmazonEKSClusterRole
    * Click 'Create role'
![pics](/pics/eks-role.png)  

* Go to EKS console to create a cluster (https://console.aws.amazon.com/eks/home#/clusters)
    * Click 'Add cluster' and select create
    * Input parameter as follows
        * Name: eksworkshop-eksctl
        * Kubernetes version: 1.24
        * Cluster service role: myAmazonEKSClusterRole
        * VPC: default vpc
        * Subnets: names starting with private-*
        * Security groups: default
        * Cluster IP address family: IPv4
        * cluster endpoint access: Private
        * Control plane Logging: toggle off as is
        * Add-ons: leave everything as default
    * Wait 10-15 minutes until the cluster status is Active  
    ![pics](/pics/eks-healthy.png)  

* Next step creating a node group, again we need to create a node IAM role for it
    * Follw the instruction on this guide (https://docs.aws.amazon.com/eks/latest/userguide/create-node-role.html#create-worker-node-role)
        * Permission policies: AmazonEKSWorkerNodePolicy, AmazonEC2ContainerRegistryReadOnly, AmazonEKS_CNI_Policy  
        ![pics](/pics/node-role-policy.png)  

    * Go to EKS console (https://console.aws.amazon.com/eks/home#/clusters) and click on the cluster we created in previous section
        * Click on Compute tab and choose 'Add node group'
            * name: nodegroup-1
            * Node IAM role: a role name that we created in latest bullet.
            * Leave others as default
            * AMI type: Amazon Linux 2
            * Capactiy type: On-Demand
            * Instance types: m5.large
            * Disk size: 20 GiB
            * Desired/Minimum/Maximum size: 3
            * Subnets: names beginning with private-*
    * Wait 5-10 minutes until the node group status is Active  
    ![pics](/pics/node-group-active.png)  

