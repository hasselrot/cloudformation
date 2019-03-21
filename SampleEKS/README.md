# Sample EKS Stack
These Cloudformation templates aim to make deployment of a production grade EKS cluster as simple as possible.
While still supporting features like multiple groups of EKS nodes, Kubernetes Autoscaler and External DNS.

### Load balancing
These stacks are prepared for use with AWS ALB Ingress Controller.
https://aws.amazon.com/blogs/opensource/kubernetes-ingress-aws-alb-ingress-controller/

No ALB will be created by these stacks, but when you deploy the ALB Ingress controller you can control the automatic creation of ALBs.

### External DNS
These stacks are prepared for use with Kubernetes External DNS.
https://github.com/kubernetes-incubator/external-dns

### Auto scaling
These stacks are prepared for use with Kubernetes Autoscaler.
https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md

### Overview of the templates
The stacks are meant to be run in order (for the most part).
More details in each section.

* 01-eks-network-prerequisites.yaml - Creates Network
* 02-eks-cluster - Creates EKS Cluster
* 03-eks-worker-prerequisites - Creates required IAM roles and security groups
* 04-eks-nodegroup-with-autoscaling - Creates a group of EKS Nodes.
* 05-eks-nodegroup-no-autoscaling - Creates a group of EKS Nodes.

## 01-eks-network-prerequisites.yaml
This template will setup a VPC with 3 private and 3 public subnets. 

The public subnets can be used for deploying nodes that should contain frontend servers that would be accessible from the internet. Or to be used for an AWS Application Loadbalancer.
The private subnets will route any outgoing traffic through NAT Instances, and are not accessible from the internet.

### Parameters
Required parameters are the VPCCIDR and the SubnetBlocks you want to use.
The default will work for most cases.

## 02-eks-cluster.yaml
Creates the EKS Cluster.

### Parameters
You need to provide:
* The name of the stack created from 01-eks-network-prerequisites
* The name you want to give to your EKS Cluster
* The version of Kubernetes you want to run

## 03-eks-worker-prerequisites.yaml
This stack creates all the prerequisites for the EKS Nodes.

* IAM - Will create Roles for all servers. That have IAM Policies required for managing Loadbalancing, DNS and Autoscaling.
* Security groups - Allows traffic between all the nodes in the cluster, as well as to the EKS Control Plane.

### Parameters
You need to provide:
* The name of the stack created from 01-eks-network-prerequisites
* The name you gave the cluster in 02-eks-cluster
* The Hosted Zone ID of your Route53 zone that you want to control with Kubernetes External DNS (this is not required to continue the stack)

## 04-eks-nodegroup-with-autoscaling & 05-eks-nodegroup-no-autoscaling
These stacks create the AWS Autoscaling groups for the EKS Nodes.
They can both be run multiple times, if you for example want to have a group of nodes with CPU Optimized instances and one with RAM Optimized instances.

The difference between 04 and 05 is in the tagging that controls the Kubernetes Autoscaler.

04-eks-nodegroup-with-autoscaling.yaml has the following configuration on the Autoscaling group:

```
- Key: k8s.io/cluster-autoscaler/enabled
    Value: "True"
    PropagateAtLaunch: 'true'
```

### Parameters
You need to provide:
* The name of the stack created from 01-eks-network-prerequisites
* The name you gave the cluster in 02-eks-cluster
* The name of the stack created from 03-eks-worker-prerequisites


