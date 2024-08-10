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

[Amazon EKS Cluster VPC Considerations](https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html)

[Tagging Your Amazon EKS Resources](https://docs.aws.amazon.com/eks/latest/userguide/eks-using-tags.html)

[Load Balancers and Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/network-load-balancing.html)

[Terraform AWS Provider: EKS](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eks_cluster)

## Steps to create eks cluster
## Add IAM User and IAM Role to EKS
In AWS we can use IAM Users or IAM Roles as objects that represent identities. 

We will create a IAM User which have read permissions on eks. And then create on k8s a Role with read permissions and the rolebinding to map the user with the Role. 

`
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: viewer
rules:
  - apiGroups: ["*"]
    resources: ["deployments", "configmaps", "pods", "secrets", "services"]
    verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-viewer-binding
roleRef:
  kind: ClusterRole
  name: viewer
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: Group
    name: my-viewer
    apiGroup: rbac.authorization.k8s.io    
`
For the IAM User, check `add-developer-user.tf`

`Test:`
- Generate access key for the user and configure a profile: `aws configure --profile developer`, you can check if the profile is configure proporly by running `aws sts get-caller-identity --profile developer`
- Connect to the eks cluster: 
```
aws eks update-kubeconfig \
--region us-east-1 \
--name dev-korner \
--profile developer
```

### Using IAM Role
The best practices is not to use IAM User with long term credentials but use IAM Role; so we will create a IAM Role with admin permissions and bind it with default cluster-admin Role; Then we will create a IAM User manager with additional policy to allow him to assume the eks-admin IAM Role. 

```
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-admin-binding
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: Group
    name: my-admin
    apiGroup: rbac.authorization.k8s.io
```

For the IAM User, check `add-manager.tf`

`Test:`
- Generate access key for the manager user and configure a profile: `aws configure --profile manager`, you can check if the profile is configure proporly by running `aws sts get-caller-identity --profile manager`
- Assume the created Role: `aws sts assume-role --role-arn <> --role-session-name <> --profile manager`
- Create manually a profile
```
[profile eks-admin]
role_arn = <>
source_profile = manager
```
- Connect to the eks cluster: 
```
aws eks update-kubeconfig \
--region us-east-1 \
--name dev-korner \
--profile eks-admin
```
