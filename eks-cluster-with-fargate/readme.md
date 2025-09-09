# Demo Project: Create EKS Cluster with Fargate Profile

## Technologies Used
- Kubernetes  
- AWS EKS  
- AWS Fargate  

---

## Project Description
- [Create Fargate IAM Role](#create-fargate-iam-role)  
- [Create Fargate Profile](#create-fargate-profile)  
- [Deploy an Example Application to EKS Cluster Using Fargate Profile](#deploy-an-example-application-to-eks-cluster-using-fargate-profile)  

---

## Create Fargate IAM Role

To create the IAM role required for Fargate pods:

1. Open the **AWS Console** and navigate to **IAM** → **Roles**.  
2. Click **Create role**.  
3. Under **Select trusted entity**, choose **EKS service**.  
4. Select **EKS - Fargate Pod**.  
5. Click **Next**.  
6. Check the **preselected pod policy**.  
7. Click **Next**.  
8. Set **Role Name**: `eks-fargate-role`.  
9. Click **Create role**.

---

## Create Fargate Profile

To create a Fargate profile for your EKS cluster:

1. Open the **AWS Console** and navigate to **EKS** → **Clusters**.  
2. Select your cluster: `eks-cluster-test`.  
3. Go to the **Compute** tab and scroll down to **Fargate profiles**.  
4. Click **Create Fargate profile**.  

5. **Fargate Profile Configuration**:  
   - **Name**: `dev-profile`  
   - **Pod execution role**: `eks-fargate-role`  
   - Click **Next**.  

6. **Pod Selectors**:  
   - **Pod selectors**: `dev`  
   - Expand **Match Labels**:  
     - **Key**: `profile`  
     - **Value**: `fargate`  

7. Click **Next**, review the configuration, and then **Create**.

---

## Deploy an Example Application to EKS Cluster Using Fargate Profile

1. Check Compute Resources
In the **EKS Console**, under your cluster’s **Compute** tab, you can see both:  
- **Node Group**  
- **Fargate Profile**  

2. Verify Existing Pods and Nodes
Open a terminal and run:
```bash
kubectl get pods
kubectl get nodes -n kube-system
kubectl get ns
```

3. Create a new namespace *dev*:
```bash
kubectl create ns dev
```

4. Deploy the Nginx application using the Fargate profile:
```bash
kubectl apply -f nginx-config.yaml
```

5. Verify pods in the dev namespace: `kubectl get pods -n dev`
6. Check the nodes serving the dev namespace: `kubectl get nodes -n dev`
