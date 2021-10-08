1. http
2. https
3. share ingress alb

## AWS Load Balancer Controller Overview

- IP targets vs ...
- Find you the latest version from [Tags](https://github.com/kubernetes-sigs/aws-load-balancer-controller/tags) - `v2.2.4`

## Create EKS with Terraform

## Setup IAM role for service accounts

## Create an IAM OIDC provider

- Copy `OpenID Connect provider URL` from the EKS cluster `Configuration` tab
- From `IAM` section create identity provider of type `OpenID Connect`
- For `Audience`, enter `sts.amazonaws.com`
  
## Create IAM policy for the AWS Load Balancer Controller

- Download policy for the current verion you trying to install

```bash
curl -o iam-policy.json \
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.2.4/docs/install/iam_policy.json
```

- Create `AWSLoadBalancerControllerIAMPolicy` IAM policy

## Create a IAM role and ServiceAccount for the AWS Load Balancer controller

- Go to `IAM` -> `Roles` section and create a `aws-load-balancer-controller` IAM role
- Attach `AWSLoadBalancerControllerIAMPolicy` policy to the role
- Edit trust relationship, replace `aud` -> `sub`, and `sts.amazonaws.com` -> `system:serviceaccount:kube-system:aws-load-balancer-controller`
- Verify that the IAM role is correctly associated with Kubertnete service account

## Deploy AWS Load Balancer Controller

If you decide to use YAML, you also need to install cert-manager

- Add the EKS chart repo to helm

```bash
helm repo add eks https://aws.github.io/eks-charts
```

- Update Helm repo

```bash
helm repo update
```

- Create `values.yaml` to override default [variables](https://github.com/aws/eks-charts/blob/master/stable/aws-load-balancer-controller/values.yaml)

- Install Helm chart

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system -f values.yaml
```

## Clean Up

- Delete EKS cluster

```bash
eksctl delete cluster -f eks.yaml
```

- Delete `AWSLoadBalancerControllerIAMPolicy` IAM policy

- Remove EKS chart repo

```bash
helm repo remove eks
```

## Links

- [Application load balancing on Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)
- [GitHub aws-load-balancer-controller](https://github.com/kubernetes-sigs/aws-load-balancer-controller)
- [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/ingress/annotations/)

## Notes

Prerequisites
- The AWS Load Balancer Controller provisioned on your cluster
- At least two subnets in different Availability Zones.
- Security group attached to worker node must have following label
  - `Key – kubernetes.io/cluster/cluster-name`
  - `Value – shared or owned`
- The cluster-name value is for your Amazon EKS cluster. The `shared` value allows more than one cluster to use the subnet.
- Subnets may be tagged
  - `Key – kubernetes.io/cluster/cluster-name`
  - `Value – shared or owned`

- Ingress TrafficL
  - Instance mode
  - IP mode

- You can run the controller on a non-EKS cluster, for example kops or vanilla k8s. Here are the things to consider -
  - In lieu of IAM for service account, you will have to manually attach the IAM permissions to your worker nodes IAM roles
  - Ensure subnets are tagged appropriately for auto-discovery to work
  - For IP targets, pods must have IPs from the VPC subnets. You can configure amazon-vpc-cni-k8s plugin for this purpose.

- [Pod readiness gate support](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/deploy/pod_readiness_gate/)
  - example shows how it works/effective

- Create Ingress and Service

- TLS ingress

AWS load balancer controller has closer integration between aws and kubernetes vs nginx

IP mode¶
IP target mode supports pods running on AWS EC2 instances and AWS Fargate. In this mode, the AWS NLB targets traffic directly to the Kubernetes pods behind the service, eliminating the need for an extra network hop through the worker nodes in the Kubernetes cluster.

The AWS in-tree controller ignores those services resources that have the service.beta.kubernetes.io/aws-load-balancer-type annotation as external
service.beta.kubernetes.io/aws-load-balancer-type: "external"
https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/service/nlb/
