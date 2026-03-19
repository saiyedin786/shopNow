
1. Setup Tools**: docs/TOOLS-SETUP-GUIDE.md
Docker installation
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo systemctl start docker
sudo systemctl enable docker

sudo usermod -aG docker $USER
newgrp docker
docker --version
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/c16e7009-d9d9-418f-a430-4c3e17f30164" />
Kubectl installation:
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version –client
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/13b5b36a-ca17-464e-8dda-7d08a6cf51a4" />

AWS cli installation:
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/62466d63-74bd-474d-8a3b-a9e4ad6200fd" />

Eksctl installation:
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/7cc02b0b-b648-4f7b-a400-3f561af76eed" />

eksctl create cluster --name shopnow-demo-cluster --region ap-south-1 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region ap-south-1 --name shopnow-demo-cluster
kubectl get nodes
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/7b617062-0d03-4918-a36e-c80dadc45c08" />

2. AWS ECR Registry Setup
# Setup AWS credentials first
aws configure
# Enter your AWS Access Key ID, Secret Access Key, region (us-east-1), and output format (json)

# Or use environment variables
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key
export AWS_DEFAULT_REGION=us-east-1

# If above credentials are already set, run below command to verify
aws sts get-caller-identity

# Create ECR repositories either via the aws cli as mentioned below or via console (Has to be done once to create the ECR repo, skip this step when you are rebuilding the docker images):

like:

aws ecr create-repository --repository-name <your-username>-shopnow/frontend --region <region>
aws ecr create-repository --repository-name <your-username>-shopnow/backend --region <region>
aws ecr create-repository --repository-name <your-username>-shopnow/admin --region <region>

# Get login token (run this command everytime as the docker credentials are persisted only on the terminal)
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/f46d6158-f198-440a-9f4d-3b7d6dbef4eb" />

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/13cc8b26-0f71-45b0-9325-ca3605888014" />

3. Update Configurations in below mentioned files
🔧 Personalization Required
For Multi-User Kubernetes Clusters: To avoid conflicts when multiple learners use the same cluster, each user must personalize their deployment with unique identifiers.
IMPORTANT: This project contains hardcoded references that you must update with your own values:
3.1. Replace "aryan" with your username in these locations:
Ingress Paths (in both Kubernetes manifests and Helm charts):
•	kubernetes/k8s-manifests/ingress/ingress-shopnow.yaml
o	Change /aryan to /<your-username>
o	Change /aryan-admin to /<your-username>-admin
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/7ee40382-92e7-4656-9669-a928e84a4f9a" />

•	kubernetes/helm/charts/frontend/values.yaml
o	Change path: /aryan to path: /<your-username>
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/401f88fb-4c02-4881-93ce-2e034d3850d9" />

•	kubernetes/helm/charts/admin/values.yaml
o	Change path: /aryan-admin to path: /<your-username>-admin
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/a50af2fe-8faa-4443-bce5-eb887f7befe1" />

Nginx ConfigMaps
•	All references with 'aryan' to in following files:
•	kubernetes/k8s-manifests/frontend/cm-nginx.yaml
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/53a71b46-c71e-47e8-bc62-b922cdbfca2c" />

•	kubernetes/k8s-manifests/admin/cm-nginx.yaml
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/6299f5ab-7605-44a5-aa06-396e018dcf10" />

Helm Chart Nginx Configurations:
•	All references with 'aryan' in the 'nginx.config' section to in following files:
•	kubernetes/helm/charts/frontend/values.yaml
•	kubernetes/helm/charts/admin/values.yaml
Dockerfiles (Build Arguments):
•	frontend/Dockerfile
o	Change ARG USER_NAME=aryan to ARG USER_NAME=<your-username>
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/4eb83cbd-9f42-4f0a-946c-d0aea77bd1fe" />

•	admin/Dockerfile
o	Change ARG USER_NAME=aryan to ARG USER_NAME=<your-username>
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/a6f5a485-7388-4708-92d3-fe9071b7da45" />

Build Script (optional):
•	scripts/build-and-push.sh
o	Update the example usage comments that reference "aryan"
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/e905e4cd-4793-4bec-a2c8-93918a5928a4" />

3.2. ECR Repository Names - Update to your username:
•	All kubernetes/k8s-manifests/*/deployment.yaml files
•	All kubernetes/helm/charts/*/values.yaml files
•	All jenkins\Jenkinsfile.*.* files
•	Change shopnow/frontend to <your-username>-shopnow/frontend
•	Change shopnow/backend to <your-username>-shopnow/backend
•	Change shopnow/admin to <your-username>-shopnow/admin
3.3. Update Namespace on these locations:
•	kubernetes/k8s-manifests/namespace/namespace.yaml - Change namespace name
•	All files in kubernetes/k8s-manifests/*/ - Update namespace references
•	kubernetes/argocd/apps/*.yaml - Update destination namespace
•	All kubectl commands in this README - Replace shopnow-demo with your namespace
3.4. Update ArgoCD Repository URL:
•	In kubernetes/argocd/umbrella-application.yaml and all kubernetes/argocd/apps/*.yaml files:
•	Change repoURL: 'https://github.com/aryanm12/shopNow'
•	To repoURL: 'https://github.com/<your-github-username>/<your-repo-name>'
4. Kubernetes Cluster Access (Make sure to have a running Kubernetes cluster, here is an example to connect with EKS)
# For EKS cluster
aws eks update-kubeconfig --region <region> --name <your-cluster-name>

# Verify access
kubectl cluster-info
kubectl get nodes
Note: All the below mentioned kubectl commands assume that you are working with "shopnow-demo" namespace, update the namespace as per yours where ever you find "shopnow-demo".

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/e9897e1b-1624-4e11-9c2a-0f147b1113b4" />

5. Docker Registry Secret (Only required for private ECR registry)
Note: Skip this step if using public Docker Hub images or public ECR repositories.
# Create registry secret for private ECR image pulls
kubectl create ns shopnow-demo
kubectl create secret docker-registry ecr-secret --docker-server=<account-id>.dkr.ecr.us-east-1.amazonaws.com --docker-username=AWS --docker-password=$(aws ecr get-login-password --region us-east-1) --namespace=shopnow-demo

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/c844d021-bbd5-4f8f-8dbc-e6bdb15645e4" />
Cloning the repo:
git clone https://github.com/saiyedin786/shopNow.git

6. Install Pre-requisites in the Kubernetes Environment (Has to be done once per Kubernetes Cluster)
# Install metrics server (required for resource monitoring and HPA)
kubectl apply -f kubernetes/pre-req/metrics-server.yaml
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/fc75f34c-1fa5-42f3-a3c5-2ce6a051c3af" />

# Install ingress-nginx controller (for external access)
# For EKS, other cloud provider will have different file
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0-beta.0/deploy/static/provider/aws/deploy.yaml

# For local development (minikube/kind/Docker Desktop)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/kind/deploy.yaml

# Verify installations
kubectl get pods -n kube-system
kubectl get pods -n ingress-nginx
kubectl top nodes  # Should work after metrics server is running
kubectl top pods  # Should work after metrics server is running

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/418149e8-46da-43d8-802e-64e8280631cf" />
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/393abf4b-3a36-44b3-9bde-affef5fb8cbb" />
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/8597f69f-1247-4e42-a961-6e0b6838b3fa" />

# To enable Persistent Storage

# First install the EBS CSI driver as an EKS Addon

-> In the EKS Console, open your cluster → go to Add-ons → click Get more add-ons → select Amazon EBS CSI driver → click Next.
-> On the configuration page, Under Pod identity association, choose Create a new IAM role, and the console will auto-attach the AmazonEBSCSIDriverPolicy.
-> Confirm and click Create. The add-on installs, the IAM role is associated with the SA via Pod Identity, and the driver starts running.
-> Verify under Add-ons tab that the EBS CSI driver is active and under Pod identity associations tab you see the SA <-> IAM role mapping.

# Install storage class for persistent volumes
kubectl apply -f kubernetes/pre-req/storageclass-gp3.yaml

# Verify storage class installation
kubectl get storageclass

⚡ Build and Deploy the micro-services
1. Build the docker images and push it to the ECR registry created above
scripts/build-and-push.sh <account-id>.dkr.ecr.<region>.amazonaws.com/<registry-name> <tag-name-number> <your-username> 



# Example for user 'aryan' with tag 'latest' and ECR registry '975050024946.dkr.ecr.ap-southeast-1.amazonaws.com/shopnow':
./scripts/build-and-push.sh 975050024946.dkr.ecr.ap-southeast-1.amazonaws.com/shopnow latest aryan


chmod +x scripts/build-and-push.sh
scripts/build-and-push.sh 254292659362.dkr.ecr.ap-south-1.amazonaws.com/ismail-shopnow latest ismail
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/64d34573-2a85-4760-a010-739097e5b514" />

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/4ccbfd65-d4b3-4338-8687-a5c77b657bac" />
2. Choose Your Deployment Method
Option A: Raw Kubernetes Manifests
kubectl apply -f kubernetes/k8s-manifests/namespace/
kubectl apply -f kubernetes/k8s-manifests/database/
kubectl apply -f kubernetes/k8s-manifests/backend/
kubectl apply -f kubernetes/k8s-manifests/frontend/
kubectl apply -f kubernetes/k8s-manifests/admin/
kubectl apply -f kubernetes/k8s-manifests/ingress/
kubectl apply -f kubernetes/k8s-manifests/daemonsets-example/

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/ea66486f-677d-4276-a71e-dcb3d199d6f1" />

Option B: Helm Charts
helm upgrade --install mongo kubernetes/helm/charts/mongo -n shopnow-demo --create-namespace
helm upgrade --install backend kubernetes/helm/charts/backend -n shopnow-demo
helm upgrade --install frontend kubernetes/helm/charts/frontend -n shopnow-demo
helm upgrade --install admin kubernetes/helm/charts/admin -n shopnow-demo
Option C: ArgoCD GitOps
# Install ArgoCD first
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Create target namespace
kubectl create namespace shopnow-demo

# Deploy applications
kubectl apply -f kubernetes/argocd/umbrella-application.yaml

# Check all ArgoCD application status:
kubectl get applications -n argocd
3. Create users in MongoDB after the mongodb pods are healthy
# check the status of the mongo-0 pods 
kubectl get pods -n shopnow-demo

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/f98f91c3-b14b-4215-b99e-530733c55c8d" />

# if mongo-0 pod is healthy, then run following command to create a user for the backend to connect
# user credentials should be same as mentioned in the backend secrets-db.yaml file
# First exex into the pods
kubectl -n shopnow-demo exec -it mongo-0 -- mongosh

# Run below commands
use admin;
db.createUser({
  user: 'shopuser',
  pwd: 'ShopNowPass123',
  roles: [
    { role: 'readWrite', db: 'shopnow' },
    { role: 'dbAdmin', db: 'shopnow' }
  ]
});

Exit
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/fa65248a-099b-45f0-b282-33f4975f9687" />

# Restart backend deployment
kubectl rollout restart deploy backend -n shopnow-demo
3. Check the resources deployed
# Check Pods
kubectl get pods -n shopnow-demo

# Check Deployment
kubectl get deploy -n shopnow-demo

# Check Services
kubectl get svc -n shopnow-demo

# Check daemonsets
kubectl get daemonsets -n shopnow-demo

# Check statefulsets
kubectl get statefulsets -n shopnow-demo

# Check HPA
kubectl get hpa -n shopnow-demo

# Check all of the above at once
kubectl get all -n shopnow-demo

# Check configmaps
kubectl get cm -n shopnow-demo

# Check secrets
kubectl get secrets -n shopnow-demo

# Check ingress
kubectl get ing -n shopnow-demo

# Sequence to debug in case of any issue with the pods
kubectl get pods -n shopnow-demo
kubectl describe pod backend-746cc99cd-cqrgf -n shopnow-demo # Assuming that pod backend-746cc99cd-cqrgf has an error
kubectl logs backend-746cc99cd-cqrgf -n shopnow-demo --previous # If no details are found in the above command or if details like liveness probe failed are coming

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/7979f50c-f726-40fc-aba8-bbc7e002dd2e" />
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/29db6c6b-e995-4f46-bc53-7ee733afb6b0" />
Access the Apps
•	Customer App → http:///
•	Admin Dashboard → http:///-admin
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/ba548a8f-440f-4813-aabe-8e4b27e7de7a" />
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/9c831bc6-677c-4f71-bec2-6caabc059c46" />































