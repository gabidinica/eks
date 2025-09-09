# Demo Project

## Create AWS EKS cluster with a Node Group

### Technologies Used
- Kubernetes  
- AWS EKS  

---

## Project Description
- [Configure necessary IAM Roles](#configure-necessary-iam-roles)  
- [Create VPC with CloudFormation Template for Worker Nodes](#create-vpc-with-cloudformation-template-for-worker-nodes)  
- [Create EKS cluster (Control Plane Nodes)](#create-eks-cluster-control-plane-nodes)  
- [Create Node Group for Worker Nodes and attach to EKS cluster](#create-node-group-for-worker-nodes-and-attach-to-eks-cluster)  
- [Configure Auto-Scaling of worker nodes](#configure-auto-scaling-of-worker-nodes)  
- [Deploy a sample application to EKS cluster](#deploy-a-sample-application-to-eks-cluster)  

---

## Configure necessary IAM Roles

Role will be assigned to the EKS cluster. Follow these steps:

1. Login into your **AWS Console**.  
2. Navigate to **IAM** → **Roles**.  
3. Click **Create Role**.  
4. Under **Service or use case**, search for **EKS** and select **EKS Cluster** option.  
5. Click **Next**.  
6. Ensure the policy attached to this role is **AmazonEKSClusterPolicy**.  
7. Click **Next**.  
8. Set **Role name** as `eks-cluster-role`.  
9. Click **Create Role**. 

---

## Create VPC with CloudFormation Template for Worker Nodes

The worker nodes for the Kubernetes cluster will run inside a VPC managed in **your AWS account**.  
EKS itself (the control plane) runs in a separate VPC managed by AWS outside your account.  

Follow these steps:

1. In the **AWS Console**, go to **VPC**.  
   - A default VPC already exists, but we’ll create a **new VPC** for worker nodes.  
2. Open the AWS documentation for VPC creation:  
   👉 [Creating a VPC for your Amazon EKS cluster](https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html)  
3. Use the official CloudFormation template for **private and public subnets**:  
   👉 [amazon-eks-vpc-private-subnets.yaml](https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml)  
4. Go to **CloudFormation** and click **Create stack**:  
   - Select **Choose an existing template**.  
   - Paste the Amazon S3 template URL above.  
   - Click **Next**.  
5. On the **Stack details** page:  
   - Enter **Stack Name**: `eks-worker-node-vpc-stack`.  
   - Click **Next**.  
6. Leave everything else as default and click **Next** again.  
7. Review the configuration and click **Submit**.  
8. Once the stack is created, go to the **Outputs** tab.  
   - You will find **3 important components** required later for creating the EKS cluster:  
     - VPC ID  
     - Public Subnet IDs  
     - Private Subnet IDs  

---

## Create EKS cluster (Control Plane Nodes)

Follow these steps to create the EKS control plane:

1. Open a new **AWS Console tab** and search for **EKS**.  
2. Click **Create Cluster**.  
3. Select **Custom Configuration**.  
   - Toggle **off** **EKS Auto Mode**.  
4. **Cluster Configuration**:  
   - Cluster name: `eks-cluster-test`  
   - For **Cluster IAM Role**, click in the field and select the IAM role you created earlier (`eks-cluster-role`).  
5. **Cluster Access**:  
   - Select **Allow cluster administrator access**.  
   - Enable both **EKS API** and **ConfigMap**.  
   - Click **Next**.  
6. **Networking**:  
   - For **VPC**, choose the VPC created with CloudFormation.  
   - For **Subnets**, open the dropdown and select **all subnets**.  
   - For **Security Group**, select the **EKS worker node security group** (not the default one).  
   - Cluster endpoint access: **Private and Public**.  
   - Click **Next**.  
7. **Add-ons**:  
   - Leave the default ones checked.  
   - Click **Next**.  
8. **Review and Create**:  
   - Review all the details.  
   - Click **Create** to launch the EKS cluster.  

###Connect to EKS Cluster Locally with kubectl
After the EKS cluster is created, connect to it from your local machine using **kubectl**.

1. Open a terminal on your computer.  
2. Verify AWS CLI configuration:  
```bash
aws configure list
```

3. Update your kubeconfig with the new EKS cluster:
```bash
aws eks update-kubeconfig --name eks-cluster-test
```

4. View the generated kubeconfig file: `cat ~/.kube/config`
5. Check if nodes are available: `kubectl get nodes`
6. List namespaces: `kubectl get ns`
7. Verify cluster info and endpoint: `kubectl cluster-info`
> The endpoint should match the one shown in the Overview tab of the EKS console.

---

## Create Node Group for Worker Nodes and attach to EKS cluster

### Create EC2 IAM Role for Node Group

The kubelet running on each worker node (EC2 instance) requires permissions to connect with the EKS cluster, pull images from ECR, and allow pod networking.

Follow these steps:

1. In the **AWS Console**, go to **IAM → Roles**.
2. Click *Create Role*.
3. In the dropdown, select *EC2* as the use case and click *Next*.
4. From the policies list, attach the following:
	- AmazonEKSWorkerNodePolicy (permissions for worker nodes)
	- AmazonEC2ContainerRegistryReadOnly (access to ECR for pulling container images)
	- AmazonEKS_CNI_Policy (networking so pods can communicate with each other)
5. Click Next.
6. Set *Role name* as *eks-node-group-role*.
7. Click Create Role.

### ## Create Node Group for Worker Nodes and attach to EKS cluster

After the EKS cluster is created, follow these steps to create a Node Group:

1. In the **AWS Console**, open the **EKS cluster** you just created.  
2. Go to the **Compute** tab.  
3. Click **Add Node Group**.  
4. **Node Group Configuration**:  
   - Name: `eks-node-group`  
   - Check that the **IAM Role** is the one we just created (`eks-node-group-role`).  
   - Click **Next**.  
5. **Set Compute and Scaling Configuration**:  
   - Leave all default options.  
   - **Instance type**: `t3.small`  
6. **Node Group Scaling Configuration**:  
   - Desired size: `2`  
   - Minimum size: `2`  
   - Maximum size: `2`  
   - Click **Next**.  
7. **Remote Access Configuration**:  
   - Toggle **on** Configure remote access to nodes.  
   - Select your **docker-server key pair** or create a new one.  
   - For **Allow remote access from**, select **All**.  
   - Click **Next**.  
8. Review the **summary** and click **Create**.  
9. After creation, go to the **EC2 Dashboard** and verify that **2 EC2 instances** have been launched.  
10. Go back to your terminal and verify nodes with kubectl:  
```bash
kubectl get nodes
```

> You should see the 2 EC2 instances that were just launched as part of your Node Group.

---

## Configure Auto-Scaling of worker nodes

To enable auto-scaling for your EKS worker nodes, follow these steps:

### 1. Create IAM Policy for Cluster Autoscaler
1. Open a new **AWS Console tab** and go to **IAM** → **Policies**.  
2. Click **Create Policy**.  
3. Use the policy from the attached file: `custom-policy.json`.  
4. Click **Next**.  
5. Enter **Policy Name**: `ClusterAutoscalerPolicy`.  
6. Click **Create Policy**.  

### 2. Configure OpenID Connect (OIDC) Provider
1. Go to **EKS** → select your cluster → **Details** tab.  
2. Copy the **OpenID Connect provider URL**.  
3. Go back to **IAM** → **Identity Providers** → click **Add provider**.  
4. Select **OpenID Connect** as the provider type.  
5. Paste the copied **Provider URL**.  
6. In **Audience**, type: `sts.amazonaws.com` (represents the AWS Security Token Service).  
7. Click **Add provider**.  

### 3. Create IAM Role for Cluster Autoscaler
1. In **IAM**, go to **Roles** → click **Create Role**.  
2. Choose **Web Identity** as the trusted entity type.  
3. For **Web Identity Provider**, select the **OIDC provider** you just created.  
4. For **Audience**, type: `sts.amazonaws.com`.  
5. Click **Next**.  
6. Attach the policy **ClusterAutoscalerPolicy** created earlier.  
7. Click **Next**.  
8. Enter **Role Name**: `EKSServiceAccountRole`.  
9. Click **Create Role**.  

## Deploy Cluster Autoscaler on EKS

Follow these steps to deploy the Cluster Autoscaler and link it to the IAM role:

### 1. Download the Cluster Autoscaler YAML
Open the following URL in a browser or use `curl` to download:  
   [Cluster Autoscaler YAML for AWS](https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml)  

---

### 2. Update the YAML for ServiceAccount
1. Open the downloaded YAML file in a text editor.  
2. Locate the **ServiceAccount** section.  
3. Under `namespace`, add the following **annotation**:  
```yaml
annotations:
  eks.amazonaws.com/role-arn: ROLE_ARN_FROM_AWS
```

> Replace ROLE_ARN_FROM_AWS with the ARN of the IAM role created earlier:
 - Go to IAM → Roles → EKSServiceAccountRole.
 - Copy the Role ARN and paste it into the YAML.
4. Scroll down to the Deployment section. Under annotations, add: `cluster-autoscaler.kubernetes.io/safe-to-evict: "false"`
5. Locate the *command* section in the *Deployment* and replace `<YOUR CLUSTER NAME>` with your cluster name: `--cluster-name=eks-cluster-test`
6. Under the containers section, update the image version to match your Kubernetes version:
	- Go to EKS Console → select eks-cluster-test → check the Kubernetes version.
	- Replace the image tag with the corresponding version from your cluster.

After updating the YAML file, apply it to your EKS cluster:
- Open your terminal and run:
```bash
kubectl apply -f cluster-autoscaler-autodiscover.yaml
```

- Check that the Cluster Autoscaler deployment is running:
```bash
kubectl get deployment -n kube-system cluster-autoscaler
```

- List all pods in the kube-system namespace to verify the Cluster Autoscaler pods are active:
```bash
kubectl get pods -n kube-system
```

> You should see the Cluster Autoscaler pods running along with other system pods.

- Check which node the pod is running on
Replace `<pod-name>` with the actual Cluster Autoscaler pod name:
```bash
kubectl get pod <pod-name> -n kube-system -o wide
```

- View logs of the Cluster Autoscaler pod
```bash
kubectl logs -n kube-system <pod-name>
```

> This will display the logs of the Cluster Autoscaler, including scaling actions and any errors.

---

## Deploy a sample application to EKS cluster

We will deploy the Nginx application using the provided `nginx-config.yaml` file.

1. Apply the Nginx configuration
```bash
kubectl apply -f nginx-config.yaml
```

2. Check the status of the deployed pods:
`kubectl get pods`

3. Check the services to get the Load Balancer information:
`kubectl get svc`

4. Access the Nginx Application
	- Go to AWS Console → EC2 → scroll down to Load Balancers.
	- Click on the Load Balancer created for the Nginx service.
	- Copy the DNS Name.
	- Open a new browser tab and paste the DNS Name.
> You should see the “Welcome to Nginx” page.

5. In AWS console, edit the Autoscaler and set:
   - Desired size: `1`  
   - Minimum size: `1`  
   - Maximum size: `3`

### Scale Up Replicas
1. Edit the Nginx deployment:

```bash
kubectl edit deployment nginx
```

2. Update the replicas field to 20:
```bash
spec:
  replicas: 20
```

3. Save and exit.
4. Verify pods: `kubectl get pods`
> Some pods may show as Pending if there are not enough nodes.

5. Check current nodes:`kubectl get nodes`
> You should see 3 EC2 instances (or the number you have configured). You can also verify the instances in the Load Balancer page in AWS under EC2 page.

### Scale Down Replicas
1. Edit the Nginx deployment:

```bash
kubectl edit deployment nginx
```

2. Update the replicas field to 1:
```bash
spec:
  replicas: 1
```

3. Save and exit.
4. Verify pods: `kubectl get pods`
5. Check nodes: `kubectl get nodes`
