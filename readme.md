# AWS EKS 

## Creating VPC
EKS require to create multiple subnets in at least 2 differenets AZ ([docs]("https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html"))
- private subnets: k8s nodes
- public sunets: NLB/ALB

We will create an: 
- `Internet Gateway` which will provide internet access to any VM that have public IP
- `NAT Gateway` wich allow private instances to have access to internet

`NOTE:`
When creating a Virtual Private Cloud (VPC) for Amazon Elastic Kubernetes Service (EKS), specific tags are required to ensure proper integration and functioning of the resources within the EKS cluster:
- `kubernetes.io/role/internal-elb`: This tag is critical for internal Elastic Load Balancer (ELB) functionality within the EKS cluster. When you create internal load balancers (i.e., load balancers that are not exposed to the internet), Kubernetes uses this tag to associate the load balancers with the subnets in the VPC. It essentially tells Kubernetes that this subnet can be used for internal load balancers, facilitating the internal traffic routing within the cluster.
- `kubernetes.io/cluster/${local.env}-${local.eks_name}`: This tag is used by EKS to identify which resources (such as subnets, security groups, etc.) belong to a specific EKS cluster. The value "owned" indicates that the cluster owns these resources and should manage them accordingly. This is essential for EKS to correctly configure and manage the networking and security settings for the cluster.
- `kubernetes.io/role/elb`: This tag indicates that the subnet can be used for external load balancers, which are publicly accessible from the internet. It allows Kubernetes to identify which subnets are eligible for deploying internet-facing load balancers.

[Amazon EKS Cluster VPC Considerations]("https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html")

[Tagging Your Amazon EKS Resources]("https://docs.aws.amazon.com/eks/latest/userguide/eks-using-tags.html")

[Load Balancers and Amazon EKS]("https://docs.aws.amazon.com/eks/latest/userguide/network-load-balancing.html")

[Terraform AWS Provider: EKS]("https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eks_cluster")

