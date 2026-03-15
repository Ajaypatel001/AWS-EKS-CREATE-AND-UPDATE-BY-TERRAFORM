# AWS EKS Cluster Upgrade Guide

This guide explains how to upgrade an Amazon EKS cluster including the
control plane, node groups, and addons while ensuring minimal downtime.

------------------------------------------------------------------------

## Step 1: Check Existing Cluster Version

First, check the current Kubernetes version of the EKS cluster.

``` bash
aws eks describe-cluster --name project-eks --region us-east-1 --query "cluster.version"
```

------------------------------------------------------------------------

## Step 2: Cordon Existing Nodes

Before upgrading, cordon the old nodes so that no new pods are scheduled
on them.

``` bash
kubectl cordon <old-node-name>
```

Example:

``` bash
kubectl cordon ip-10-0-3-8.ec2.internal
kubectl cordon ip-10-0-4-252.ec2.internal
```

------------------------------------------------------------------------

## Step 3: Upgrade EKS Control Plane

Upgrade the Kubernetes version of the EKS control plane.

``` bash
aws eks update-cluster-version --region us-east-1 --name project-eks --kubernetes-version 1.32
```

Wait until the upgrade completes before proceeding.

------------------------------------------------------------------------

## Step 4: Upgrade Node Groups

After upgrading the control plane, upgrade the worker nodes using
rolling updates.

``` bash
aws eks update-nodegroup-version --cluster-name project-eks --nodegroup-name eks-node-group --kubernetes-version 1.32 --region us-east-1
```

------------------------------------------------------------------------

## Step 5: Upgrade EKS Addons

Important addons include:

-   CoreDNS
-   kube-proxy
-   VPC CNI
-   AWS EBS CSI Driver

### Upgrade VPC CNI

``` bash
aws eks update-addon --cluster-name project-eks --addon-name vpc-cni
```

### Upgrade CoreDNS

``` bash
aws eks update-addon --cluster-name project-eks --addon-name coredns
```

### Upgrade kube-proxy

``` bash
aws eks update-addon --cluster-name project-eks --addon-name kube-proxy
```

### Upgrade EBS CSI Driver

``` bash
aws eks update-addon --cluster-name project-eks --addon-name aws-ebs-csi-driver
```

------------------------------------------------------------------------

## Node Upgrade Workflow

Old Node\
↓\
Drain Node\
↓\
Replace with New Node\
↓\
Reschedule Pods

------------------------------------------------------------------------

## Resume Points -- EKS Cluster Upgrade

-   Planned and executed AWS EKS cluster version upgrades while
    minimizing downtime.
-   Managed node group upgrades ensuring compatibility with cluster
    versions.
-   Implemented default EKS addons like CoreDNS, kube-proxy, VPC CNI and
    EBS CSI driver.
-   Used rolling node updates to maintain service availability.
-   Automated cluster upgrades using Terraform best practices.
-   Monitored cluster health and application stability using AWS Console
    and kubectl.

------------------------------------------------------------------------

## Interview Explanation

I have experience performing EKS cluster upgrades using multiple
approaches. I upgraded the control plane and worker nodes using AWS
console and CLI while ensuring minimal downtime. I also automated
cluster upgrades using Terraform, managing cluster versions, node
groups, and addons like CoreDNS, kube-proxy, VPC CNI, and EBS CSI
driver.

By using rolling updates, monitoring cluster health, and validating
system pods, I ensured upgrades were safe, predictable, and production
ready.
