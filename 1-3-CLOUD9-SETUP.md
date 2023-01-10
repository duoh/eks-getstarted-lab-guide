## Set up Cloud9
Cloud9 is a Cloud IDE provided by AWS. We are going to use this as a workstation for this hands-on workshop.  
To allow Cloud9 can talk with a private EKS api endpoint, we need to place Cloud9 instance in the private subnet.  
Moreover, The instance role must be the same one as used to create EKS console.  
Last, It is require to associate one more security group, default. So Cloud9 can talk to EKS API server endpoint.  
* Go to Cloud9 console (https://console.aws.amazon.com/cloud9control)
    * Click on 'Create environment' and input the following parameters
        * Name: eks-workstation
        * Environment type: New EC2 instance
        * Instance type: t3.small
        * Platform: Amazon Linux 2
        * Timeout: Never
        * Connection: AWS Systems Manager (SSM)
        * VPC: default VPC
        * Subnet: one of private subnets that we created in NETWORK-SETUP part
    * Wait 5 minutes until Cloud9 setup complete
    ![pics](/pics/cloud9-successful.png)  
* Click on Open on Cloud9 IDE column to access the IDE
* Install necessary tools on Cloud9 IDE by following commands
    * Install eksctl
    ```
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    sudo mv -v /tmp/eksctl /usr/local/bin
    ```
    * Install kubectl
    ```
    curl -LO https://dl.k8s.io/release/v1.24.9/bin/linux/amd64/kubectl
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    kubectl version --client
    ```
    * Install awscli
    ```
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    ```
    * Install jq, envsubst, bash-completion and yq
    ```
    sudo yum -y install jq gettext bash-completion moreutils
    echo 'yq() {
    docker run --rm -i -v "${PWD}":/workdir mikefarah/yq "$@"
    }' | tee -a ~/.bashrc && source ~/.bashrc
    ```
    * Verify the binaries in the path
    ```
    for command in eksctl kubectl jq envsubst aws
    do
        which $command &>/dev/null && echo "$command in path" || echo "$command NOT FOUND"
    done
    ```
    * Enable kubectl bash_completion
    ```
    kubectl completion bash >>  ~/.bash_completion
    . /etc/profile.d/bash_completion.sh
    . ~/.bash_completion
    ```
* Change the default instance role of Cloud9 to Teamrole instance role, go to EC2 console (https://console.aws.amazon.com/ec2/)
    * Click on Instances (running), and tick on checkbox of an instance whose name contains aws-cloud9-*
    * Click on Actions, then select Security, and choose 'Modify IAM role'
    * Select IAM role to TeamRoleInstanceProfile  
    ![pics](/pics/cloud9-iamrole.png)  
    * Click on 'Update IAM role'
* Associate a default security group with Cloud9 instance, go to EC2 console (https://console.aws.amazon.com/ec2/)
    * Click on Instances (running), and tick on checkbox of an instance whose name contains aws-cloud9-*
    * Click on Actions, then select Security, and choose 'Change security groups'
    * Select default and click on 'Add security group'
    ![pics](/pics/cloud9-sg.png)  
    * Click on Save
* Verify if we can access EKS API server endpoint on Cloud9 by running the following commands
    * Save often-used values in ACCOUNT_ID and AWS_REGOIN environment variables
    ```
    export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
    export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
    ```
    * Update kube configuration, replace ${cluster-name} with EKS cluster name you created in EKS-SETUP part
    ```
    aws eks update-kubeconfig --name ${cluster-name}
    ```
    * Test if we can talk to the cluster
    ```
    kubectl get svc -A
    kubectl get node
    ```
    * Output as following
    ![pics](/pics/kubectl-svc-node.png)