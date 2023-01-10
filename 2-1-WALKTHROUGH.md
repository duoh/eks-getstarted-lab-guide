## Set up application
* Understand application architecture and clone the repository onto Cloud9 by following the guide on  
    (https://catalog.workshops.aws/eks-immersionday/en-US/aboutworkshopapp)
* Install Helm by following the guide on (https://catalog.workshops.aws/eks-immersionday/en-US/helm#installation-and-setup)
* Deploy and test application with Helm Chart by following the guide on  
    (https://catalog.workshops.aws/eks-immersionday/en-US/helm/deploy#deploy-the-helm-chart)

## Intro to RBAC
* Follow the guide on (https://catalog.workshops.aws/eks-immersionday/en-US/rbac)

## IAM Roles for Service Accounts
* Follow the guide on (https://catalog.workshops.aws/eks-immersionday/en-US/irsa)

## Services and Ingress
Before we begin, the guide will use auto-discovery feature of ALB controller to create on public/private subnets without specifying.  
* Add the appropriate tags on subnets to differentiate between public and private subnets
    * For private subnets tags with EMPTY value:
        * key: kubernetes.io/role/internal-elb
    * For public subnets tags with EMPTY value:
        * key: kubernetes.io/role/elb
* Follow ClusterIP, Headless, NodePort, LoadBalancer and Ingress sections in the guide on (https://catalog.workshops.aws/eks-immersionday/en-US/services-and-ingress)
* In Reachability Tests of NodePort section (https://catalog.workshops.aws/eks-immersionday/en-US/services-and-ingress/nodeport#5.-reachability-tests)
    * Add an inbound rule for eks-cluster-sg-* for all traffic from a default security group rather than from 0.0.0.0/0

## Observability
In this topic we focus on Container Insight, Cloudwatch Container logs and AWS X-Ray Tracing (skipping Prometheus Metrics).  
* Follow the guide on (https://catalog.workshops.aws/eks-immersionday/en-US/observability/setup#enable-amazon-cloudwatch-container-insights)
* "To deploy the X-Ray DaemonSet" guide in XRay Trace section is failed due to rbac.authorization.k8s.io/v1beta1 deprecated.  
    Run the following instruction instead  
    ```
    wget https://eksworkshop.com/intermediate/245_x-ray/daemonset.files/xray-k8s-daemonset.yaml
    sed -i 's/rbac.authorization.k8s.io\/v1beta1/rbac.authorization.k8s.io\/v1/' xray-k8s-daemonset.yaml
    kubectl apply -f xray-k8s-daemonset.yaml
    ```

## Autoscaling
In this topic we follow 'Install Kube-ops-view', 'Horizontal Pod AutoScaler (HPA)' and  'Cluster Autoscaler (CA)' sections in the guide on (https://catalog.workshops.aws/eks-immersionday/en-US/autoscaling).  
* In 'Install kube-ops-view' section (https://catalog.workshops.aws/eks-immersionday/en-US/autoscaling/install-kube-ops-view#install-kube-ops-view), follow the following command instead
```
helm repo add christianknell https://christianknell.github.io/helm-charts
helm install kube-ops-view christianknell/kube-ops-view  --set service.type=LoadBalancer --set rbac.create=True
```
* When verifying the result in 'Deploy the Metrics Server' section (https://catalog.workshops.aws/eks-immersionday/en-US/autoscaling/deploy-hpa#deploy-the-metrics-server), the failed output is shown as below 
```
{
  "conditions": [
    {
      "lastTransitionTime": "2023-01-10T11:39:33Z",
      "message": "endpoints for service/metrics-server in \"kube-system\" have no addresses with port name \"https\"",
      "reason": "MissingEndpoints",
      "status": "False",
      "type": "Available"
    }
  ]
}
```
* To fix, add --kubelet-insecure-tls=true arg in spec:containers:args of metrics-server Deployment
```
kubectl edit deploy metrics-server -n kube-system
```
The example of fix  
![pics](/pics/hpa-fixed.png)  

* In 'IAM roles for service accounts' section (https://catalog.workshops.aws/eks-immersionday/en-US/autoscaling/deploy-ca#iam-roles-for-service-accounts), add "ec2:DescribeInstanceTypes" action in k8s-asg-policy.json
```
mkdir ~/environment/cluster-autoscaler

cat <<EoF > ~/environment/cluster-autoscaler/k8s-asg-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:DescribeTags",
                "autoscaling:SetDesiredCapacity",
                "autoscaling:TerminateInstanceInAutoScalingGroup",
                "ec2:DescribeLaunchTemplateVersions",
                "ec2:DescribeInstanceTypes"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
EoF

aws iam create-policy   \
  --policy-name k8s-asg-policy \
  --policy-document file://~/environment/cluster-autoscaler/k8s-asg-policy.json
```