# Kubernetes and CI/CD Demo Projects

This repository contains multiple demo projects for deploying applications to Kubernetes clusters using **AWS EKS**, **Linode LKE**, Jenkins pipelines, and private container registries.

## Projects

1. **[Create AWS EKS cluster with a Node Group](https://github.com/gabidinica/eks/tree/main/eks-with-a-node-group)**  
   Step-by-step guide to create an EKS cluster with worker nodes.

2. **[Create EKS cluster with Fargate profile](https://github.com/gabidinica/eks/tree/main/eks-cluster-with-fargate)**  
   Deploy an EKS cluster using AWS Fargate to run serverless Kubernetes pods.

3. **[Create EKS cluster with eksctl](https://github.com/gabidinica/eks/tree/main/eks-cluster-with-eksctl)**  
   Quickly create an EKS cluster using the `eksctl` CLI tool.

4. **[CD - Deploy to EKS cluster from Jenkins Pipeline](https://github.com/gabidinica/eks/tree/main/eks-cluster-from-jenkins-pipeline)**  
   Integrate Jenkins CI/CD pipeline to deploy applications to EKS clusters.

5. **[CD - Deploy to LKE cluster from Jenkins Pipeline](https://github.com/gabidinica/eks/tree/main/lke-cluster-from-jenkins-pipeline)**  
   Use Jenkins pipelines to deploy applications to Linode Kubernetes Engine (LKE) clusters.

6. **[Complete CI/CD Pipeline with EKS and private DockerHub registry](https://github.com/gabidinica/eks/tree/main/ci-cd-with-eks-dockerhub)**  
   Build a complete CI/CD pipeline that pushes Docker images to a private DockerHub repository and deploys to EKS.

7. **[Complete CI/CD Pipeline with EKS and AWS ECR](https://github.com/gabidinica/eks/tree/main/ci-cd-with-eks-and-ecr)**  
   Full CI/CD pipeline using AWS ECR as the private Docker registry for deployments to EKS.

