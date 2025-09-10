# Demo Project: Complete CI/CD Pipeline with EKS and AWS ECR

## Technologies Used
- Kubernetes  
- Jenkins  
- AWS EKS  
- AWS ECR (Elastic Container Registry)  
- Java  
- Maven  
- Linux  
- Docker  
- Git  

---

## Project Description
- Create a **private AWS ECR Docker repository**  
- Adjust Jenkinsfile to **build and push Docker images to AWS ECR**  
- Integrate deployment to Kubernetes cluster in the CI/CD pipeline using the AWS ECR private registry  
-  Complete CI/CD Project Configuration
a. **CI Step**: Increment application version  
b. **CI Step**: Build artifact for Java Maven application  
c. **CI Step**: Build and push Docker image to **AWS ECR**  
d. **CD Step**: Deploy new application version to the **EKS cluster**  
e. **CD Step**: Commit the version update back to the repository  

---

## Create Private AWS ECR Docker Repository

1. Open the **AWS Management Console**.  
2. Navigate to **ECR → Get Started**.  
3. Select **Private Repository**.  
4. Set **Repository Name**: `java-maven-app`.  
5. Click **Create Repository**.  

✅ Your private AWS ECR repository is now ready to store Docker images.  

--- 

## Create AWS ECR Credentials in Jenkins

1. In the AWS ECR console, click **View push commands** for your repository.  
2. Get the ECR login password in your local terminal:
```bash
aws ecr get-login-password --region eu-central-1
```

> This will output the password needed to authenticate Docker with AWS ECR.

3. Add the credentials in Jenkins:
- Go to Manage Jenkins → Credentials → Global credentials → Add Credentials.
- Set the following:
	- Kind: Username with password
	- ID: ecr-credentials
	- Username: AWS
	- Password: Paste the password from the terminal
	- Click Add.
> ✅ Jenkins now has credentials to push Docker images to your private AWS ECR repository.

## Create Kubernetes Secret for AWS ECR

This secret allows your EKS cluster to pull images from the private AWS ECR repository.

### 1. Verify Existing Secrets
```bash
kubectl get secret
```

### 2. Create Docker Registry Secret
```bash
kubectl create secret docker-registry aws-registry-key \
  --docker-server=<repo-url-from-aws> \
  --docker-username=AWS \
  --docker-password=<copy-paste-password-from-terminal>
```

**Notes**
> <repo-url-from-aws> → Found in AWS ECR console under Repository URI.
> <copy-paste-password-from-terminal> → Output from: aws ecr get-login-password --region eu-central-1

Reference this secret in your deployment.yaml under imagePullSecrets:
```bash
spec:
  template:
    spec:
      imagePullSecrets:
        - name: ecr-registry-key
```

Jenkinsfile is updated, you can check it in the file attached.

## Execute Jenkins Pipeline for AWS ECR Deployment

### 1. Commit Changes
Commit your updated Jenkinsfile to your Git repository:

```bash
git add Jenkinsfile
git commit -m "Update build and deploy steps for AWS ECR"
git push origin <branch-name>
```

### 2. Trigger Jenkins Pipeline
 - Go to Jenkins → Jenkins-jobs → java-maven-app.
 - Run the pipeline for the updated branch.

### 3. Verify Docker Image in AWS ECR:
  - Open AWS Management Console → ECR → java-maven-app.
  - Check that the Docker image is tagged correctly (e.g., build number or version).

### 4. Check the pods running in your EKS cluster:
```bash
kubectl get pods
```

Inspect a specific pod for more details: `kubectl describe pod <pod_name>`
