AWS EKS Cluster Upgrade Guide

This guide explains how to upgrade an Amazon EKS cluster, including the control plane, node groups, and addons, while ensuring minimal downtime.

Step 1: Check Existing Cluster Version

First, check the current Kubernetes version of the EKS cluster.

aws eks describe-cluster \
--name project-eks \
--region us-east-1 \
--query "cluster.version"
Step 2: Cordon Existing Nodes

Before upgrading, cordon the old nodes so that no new pods are scheduled on them during the upgrade process.

kubectl cordon <old-node-name>

Example:

kubectl cordon ip-10-0-3-8.ec2.internal
kubectl cordon ip-10-0-4-252.ec2.internal

This ensures workloads are not deployed on nodes that will be replaced.

Step 3: Upgrade EKS Control Plane

Upgrade the Kubernetes version of the EKS control plane.

aws eks update-cluster-version \
--region us-east-1 \
--name project-eks \
--kubernetes-version 1.32

Wait until the upgrade completes before proceeding to the next step.

Step 4: Upgrade Node Groups

After upgrading the control plane, upgrade the worker nodes.

Recommended approach: Rolling upgrade

aws eks update-nodegroup-version \
--cluster-name project-eks \
--nodegroup-name eks-node-group \
--kubernetes-version 1.32 \
--region us-east-1

This will gradually replace the old nodes with new nodes.

Step 5: Upgrade EKS Addons

After upgrading the cluster and nodes, update the EKS addons.

Important addons include:

CoreDNS

kube-proxy

VPC CNI

AWS EBS CSI Driver

Upgrade VPC CNI
aws eks update-addon \
--cluster-name project-eks \
--addon-name vpc-cni
Upgrade CoreDNS
aws eks update-addon \
--cluster-name project-eks \
--addon-name coredns
Upgrade kube-proxy
aws eks update-addon \
--cluster-name project-eks \
--addon-name kube-proxy
Upgrade EBS CSI Driver
aws eks update-addon \
--cluster-name project-eks \
--addon-name aws-ebs-csi-driver
Node Upgrade Workflow

Old Node
↓
Drain Node
↓
Replace with New Node
↓
Reschedule Pods

This process ensures workloads continue running without downtime.

Resume Points – EKS Cluster Upgrade

Planned and executed AWS EKS cluster version upgrades while minimizing downtime for production workloads.

Managed node group upgrades using Terraform, ensuring worker nodes remained compatible with upgraded cluster versions.

Implemented default EKS addons (CoreDNS, kube-proxy, VPC CNI, EBS CSI driver, pod identity) with automated upgrade paths using Terraform.

Applied sequential addon deployment (depends_on) to reduce provisioning time and avoid AWS API throttling during upgrades.

Leveraged Terraform variables and version control to dynamically manage cluster, node group, and addon versions for repeatable deployments.

Ensured zero-impact upgrades by using rolling node updates, maintaining service availability during cluster version changes.

Automated cluster and addon upgrades using Terraform best practices, including force_update_version and version pinning when needed.

Monitored and validated post-upgrade cluster health, addon readiness, and application stability using AWS console and kubectl.

Interview Explanation

I have experience performing EKS cluster upgrades using multiple approaches.

Manually, I upgraded the control plane and worker nodes using the AWS console while monitoring system components to ensure stability and zero downtime.

Using the AWS CLI, I scripted cluster and node upgrades and validated addon and pod health to maintain operational reliability.

With Terraform, I automated the entire upgrade process by managing cluster versions, node groups, and default addons such as CoreDNS, kube-proxy, VPC CNI, and EBS CSI.

I also implemented best practices like version pinning for critical addons, rolling updates for node groups, and health checks for system pods to ensure upgrades remain predictable and low risk.

By combining automation, monitoring, and sequential deployment, I minimized downtime, maintained application availability, and created a repeatable and auditable upgrade process across multiple environments.
