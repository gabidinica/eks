# Demo Project: CD - Deploy to LKE Cluster from Jenkins Pipeline

## Technologies Used
- Kubernetes  
- Jenkins  
- Linode LKE  
- Docker  
- Linux  

---

## Project Description
- [Create Kubernetes Cluster on LKE](#create-kubernetes-cluster-on-lke)  
- [Install kubectl as Jenkins Plugin](#install-kubectl-as-jenkins-plugin)  
- [Adjust Jenkinsfile for LKE Deployment](#adjust-jenkinsfile-for-lke-deployment)  

---

## Create Kubernetes Cluster on LKE

1. Go to the Linode Cloud Console:  
👉 [https://cloud.linode.com/linodes](https://cloud.linode.com/linodes)
2. Click on **Kubernetes** → **Create Cluster**.  
3. Configure the cluster:  
   - **Cluster Label**: `test`  
   - **Region**: `Frankfurt`  
   - **HA Control Plane**: `No`  
   - **Node Pool**:  
     - **Shared CPU**: `1`  
     - **Linode 2GB** → Add it  
4. Click on **Create Cluster**.  
5. Download the kubeconfig file:  

---

## Install kubectl as Jenkins Plugin

We’re going to use the **java-maven-app** project for deployment.

### 1. Add LKE Kubeconfig as Jenkins Credentials
1. Go to Jenkins → **java-maven-app** pipeline.  
2. Navigate to **Credentials → Store → Add Credentials**.  
3. Choose **Kind**: `Secret file`.  
4. Upload the `test-kubeconfig.yaml` file.  
5. Set **ID**: `lke-credentials`.  
6. Click **Create**.  

---

### 2. Install Kubernetes CLI Plugin in Jenkins
1. Go to **Manage Jenkins → Plugins → Available Plugins**.  
2. Search for **Kubernetes CLI**.  
3. Select it and install.  
4. Restart the Jenkins server.  

---

### 3. Verify Installation
1. Go to **Manage Jenkins → Plugins → Installed Plugins**.  
2. Confirm that **Kubernetes CLI** is listed as installed.  

---

## Adjust Jenkinsfile for LKE Deployment

1. Create Feature Branch
Open the **java-maven-app** project in Visual Studio and create a new branch:

```bash
git checkout -b deploy-to-lke
```

2. Modify the Jenkinsfile by updating the *deploy stage*:
```bash
stage('deploy') {
    steps {
        script {
            echo 'deploying docker image'
            withKubeConfig([credentialsId: 'lke-credentials', serverUrl: '<cluster-endpoint>']) {
                sh 'kubectl create deployment nginx-deployment --image=nginx'
            }
        }
    }
}
```

> <cluster-endpoint> → Found in Linode Cloud Console under your Kubernetes cluster details → Kubernetes API Endpoint (without the port).

3. Commit and Push Changes to the branch.
4. Go to Jenkins → java-maven-app pipeline and click Configure.
5. In *Filter by Name*, paste the branch name: `deploy-to-lke` and Save.
6. Go to *Status* and check that branch pipeline executes.
7. Run the following command to check pods in your LKE cluster: `kubectl get pod`.
> You should see the nginx pod running.
