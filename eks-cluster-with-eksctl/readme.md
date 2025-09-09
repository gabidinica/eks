# Demo Project: Create EKS Cluster with eksctl

## Technologies Used
- Kubernetes  
- AWS EKS  
- eksctl  
- Linux  

---

## Project Description
- [Create EKS Cluster using eksctl](#create-eks-cluster-using-eksctl)  

---

## Create EKS Cluster using eksctl
`eksctl` is a command line tool that simplifies the process of creating and managing Amazon EKS clusters.  
It significantly reduces the manual effort compared to setting up an EKS cluster using only the AWS Console or CLI.  

### Install eksctl
Follow the official eksctl installation guide:  
👉 [Installation Documentation](https://eksctl.io/installation/)

You need an AWS user with programmatic access configured locally.  
Check your configuration with:

```bash
aws configure list
```

### Create EKS Cluster

Run the following command in your terminal to create the EKS cluster:
```bash
eksctl create cluster \
  --name demo-cluster \
  --version 1.27 \
  --region eu-central-1 \
  --nodegroup-name demo-nodes \
  --node-type t2.micro \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3
```

After the cluster is created, verify it using kubectl:
```bash
kubectl get nodes
kubectl get pods
```

In the AWS Console, confirm that the following resources have been created:
- IAM Roles
- VPC
- EKS Cluster 
