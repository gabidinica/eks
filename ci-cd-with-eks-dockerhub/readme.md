# Demo Project: Complete CI/CD Pipeline with EKS and Private DockerHub Registry

## Technologies Used
- Kubernetes  
- Jenkins  
- AWS EKS  
- Docker Hub (Private Registry)  
- Java  
- Maven  
- Linux  
- Docker  
- Git  

---

## Project Description
- Write Kubernetes manifest files for **Deployment** and **Service** configuration  
- Integrate a **deploy step** in the CI/CD pipeline to deploy the newly built application image from **DockerHub private registry** to the **EKS cluster**  
- Complete CI/CD Project Configuration
a. **CI Step**: Increment version  
b. **CI Step**: Build artifact for Java Maven application  
c. **CI Step**: Build and push Docker image to **DockerHub**  
d. **CD Step**: Deploy new application version to **EKS cluster**  
e. **CD Step**: Commit the version update back to the repository  

## Configure Deployment Stage for java-maven-app

We’re going to use the **java-maven-app** project and the **deploy-on-k8s** branch.  
In this folder, the necessary files have been added so you can copy them to your branch.

1.Copy the `stage("deploy")` content provided in the configuration files.
2. Navigate to your **Jenkins-jobs** repository, then open the `Jenkinsfile`.
3. Paste the `deploy` stage into the `Jenkinsfile` in the correct position within the pipeline definition.
>  Now, your pipeline will include the **deploy step**, enabling deployments to the Kubernetes cluster directly from Jenkins.  

In your `deployment.yaml` file, the Docker image is set dynamically.  

For example, in your case:

```yaml
spec:
  containers:
    - name: demo-app
      image: gabiand/demo-app:$IMAGE_NAME
```

> $IMAGE_NAME will be replaced with the actual image tag built in your CI pipeline.

Inside the **deploy stage** of your Jenkinsfile, you will apply the `deployment.yaml` and `service.yaml` manifests.  
Use `envsubst` to substitute any environment variables defined in the YAML files:  

```groovy
sh 'envsubst < kubernetes/deployment.yaml | kubectl apply -f -'
sh 'envsubst < kubernetes/service.yaml | kubectl apply -f -'
```

### Install `gettext-base` Tool on Jenkins

This is required to use `envsubst` in the deploy stage.

1. SSH into the Droplet
```bash
ssh root@<droplet-ip>
```

2. Access Jenkins Docker Container as Root:
```bash
docker exec -u 0 -it <container-id> bash
```

3. Install gettext-base: 
```bash
apt-get update
apt-get install -y gettext-base
```

4. Verify envsubst Availability: `envsubst --help`
5. Exit Container and Droplet:
```bash
exit  # exit container
exit  # exit droplet
```

### Create Docker Hub Secret for Kubernetes Deployment

Inside the **deploy stage** of your Jenkins pipeline, you need to create a Kubernetes secret for Docker Hub credentials.  

1. Verify Cluster Nodes
```bash
kubectl get nodes
```

2. Create Docker Registry Secret:
```bash
kubectl create secret docker-registry my-registry-key \
  --docker-server=docker.io \
  --docker-username=gabidin \
  --docker-password=<your-docker-password>
```

3. Verify Secret Creation:
```bash
kubectl get secret
```

> This secret allows Kubernetes to pull images from your private Docker Hub repository.It only needs to be created once per namespace.

### Execute Jenkins Pipeline for Deployment

1. Add Docker Hub Secret in Deployment
Ensure the secret `my-registry-key` is referenced in your `deployment.yaml`:
```yaml
spec:
  template:
    spec:
      imagePullSecrets:
        - name: my-registry-key
```

2. Commit all updates to your repository:
```bash
git add .
git commit -m "Add deployment secrets and update pipeline"
git push origin deploy-on-k8s
```

3. Trigger Jenkins Pipeline
- Go to Jenkins → Jenkins-jobs → java-maven-app.
- Configure the pipeline to use the branch: deploy-on-k8s.
- Trigger the pipeline run.
4. Check the pods in your EKS cluster: `kubectl get pods`
> ✅ You should see the application pods running with the correct Docker image from your private Docker Hub registry.
