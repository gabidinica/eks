# Demo Project: CD - Deploy to EKS Cluster from Jenkins Pipeline

## Technologies Used
- Kubernetes  
- Jenkins  
- AWS EKS  
- Docker  
- Linux  

---

## Project Description
- [Install kubectl and aws-iam-authenticator on Jenkins Server](#install-kubectl-and-aws-iam-authenticator-on-jenkins-server)  
- [Create kubeconfig File to Connect to EKS Cluster](#create-kubeconfig-file-to-connect-to-eks-cluster)  
- [Add AWS Credentials to Jenkins](#add-aws-credentials-to-jenkins)  
- [Extend Jenkinsfile for EKS Deployment](#extend-jenkinsfile-for-eks-deployment)  

---

## Install kubectl and aws-iam-authenticator on Jenkins Server

We’re going to use the **java-maven-app** project as the application to build and deploy in our Jenkins CD pipeline.  

👉 Project Repository: [java-maven-app](https://github.com/gabidinica/java-maven-app)

We’re going to use the previously created **demo-cluster**.  

Verify the cluster is running and currently has no pods:
```bash
kubectl get pods
```

From the previous project, Jenkins is running inside a Docker container on a **DigitalOcean Droplet**.  

### Verify Jenkins is Running
1. Make sure the droplet and the Docker container are running.  
2. Open a browser and check Jenkins at: `http://<droplet_ip>:8080`

> Connect to the Droplet
```bash
ssh root@<droplet_ip>
```

3. Check Running Docker Containers: `docker ps`
4. Run the following command to get inside the Jenkins container as the root user:
```bash
docker exec -u 0 -it <container_id> bash
```

5. Inside the Jenkins Docker container, run the following command to install **kubectl**:
```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
```

6. Check that kubectl is installed: `kubectl version`


### Install AWS IAM Authenticator

1. While inside the Jenkins Docker container, install **aws-iam-authenticator**:  
```bash
curl -Lo aws-iam-authenticator https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.6.11/aws-iam-authenticator_0.6.11_linux_amd64
```

2. Set Execute Permissions:
```bash
chmod +x ./aws-iam-authenticator
```

3. Move Binary to Local Bin:
```bash
mv ./aws-iam-authenticator /usr/local/bin
```

4. Verify installation: `aws-iam-authenticator help`
5. Then: exit

---

## Create kubeconfig File to Connect to EKS Cluster

1. Open a new file using `vim` inside the Jenkins container:
```bash
vim config
```

2. Paste the content from the `config.yaml` file.
3. Replace placeholders in the file:
 - <cluster-name> → "demo-cluster"
 - server → <endpoint-url>
	- Find the endpoint URL in the AWS Console: EKS → demo-cluster → Overview → API server endpoint
 - certificate-authority-data -> On your local machine, run: `cat ~/.kube/config` and Copy the certificate-authority-data value and paste it in the file.
4. Save and exit the editor.
5. Access the Jenkins container:
```bash
docker exec -it <container_id> bash
```

6. Go to the Jenkins home directory:
```bash
cd ~
pwd  # Should output: /var/jenkins_home
```

7. Create the .kube directory: `mkdir .kube`
8. Exit the container: `exit`
9. Copy the config file into the Jenkins container: `docker cp config <container_id>:/var/jenkins_home/.kube/`
10. Re-enter the Jenkins container: `docker exec -it <container_id> bash`
11. Verify the .kube directory exists:
```bash
cd ~
ls -a  # Check for .kube
```

12. Exit the container: `exit`
 

---

## Add AWS Credentials to Jenkins

### 1. Edit Multibranch Pipeline
1. Open Jenkins in your browser.  
2. Go to the **java-maven-app** multibranch pipeline.  
3. Click **Configure**.  
4. Update the **Source Folder** and add the new branch: `deploy-on-k8s`.  

---

### 2. Add AWS Credentials to Jenkins
1. Go to **Credentials** in Jenkins.  
2. Click **Add Credentials**.  
3. Set the following:  
   - **Kind**: Secret text  
   - **ID**: `jenkins_aws_access_key_id`  
   - **Secret**: Your actual AWS access key ID  

4. On your local terminal, verify your AWS credentials file:
```bash
cat ~/.aws/credentials
```

5. Click again on **Add Credentials**.  
6. Set the following:  
   - **Kind**: Secret text  
   - **ID**: `jenkins_aws_secret_access_key_id`  
   - **Secret**: Your actual AWS secret access key ID  

---

## Extend Jenkinsfile for EKS Deployment

Open the **java-maven-app** project in **Visual Studio Code** (or your preferred editor) make sure you are on `deploy-on-k8s` branch. 
1. Open the Jenkinsfile.
2. Update the **deploy stage** as follows:
```groovy
stage("deploy") {
    environment {
        AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key_id')
    }
    steps {
        script {
            gv.deployApp()
            sh 'kubectl create deployment nginx-deployment --image=nginx'
        }
    }
}
```

3. After updating the `Jenkinsfile` on the `deploy-on-k8s` branch:
```bash
git add Jenkinsfile
git commit -m "Update deploy stage for EKS deployment"
git push origin deploy-on-k8s
```

4. Configure Jenkins Multibranch Pipeline
- Open Jenkins and navigate to the java-maven-app multibranch pipeline.
- Click Configure.
- In Filter by Name, enter: `deploy-on-k8s`

5. After the pipeline runs successfully, check the pods in your EKS cluster:
`kubectl get pods`
