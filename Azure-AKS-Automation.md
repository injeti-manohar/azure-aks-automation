# AZURE Kubernetes Automation

# Before start this excercise,you must have running Rocky linux 8 machine.

# AZURE VM
 - create 1 rocky linux 8.5 machine in AZURE Cloud Portal
 - Note that this will work only in 'Pay-As-You-Go' Subscription and not on Free Tier
 - make sure you opened RDP,http,8080 ports for this new linux AZURE VM or you can also open any port by using port*any
 
# Steps

- Step 1:  Install AZURE CLI in rocky linux Machine
- Step 2:  First install docker in the local linux machine
- Step 3:  Download sample application
- Step 4:  Test sample application
- Step 5:  Deploy and use Azure Container Registry
- Step 6:  Install kubectl command 
- Step 7:  Deploy an Azure Kubernetes Service (AKS) cluster
- Step 8:  Run applications in Azure Kubernetes Service (AKS)


#

# Step 1: Install AZURE CLI

- Add the azure CLI Repository keys
```
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
```
- Configure the azure CLI Repository

```
sudo sh -c 'echo -e "[azure-cli]
name=Azure CLI
baseurl=https://packages.microsoft.com/yumrepos/azure-cli
enabled=1
gpgcheck=1
gpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/azure-cli.repo'
```
- Install Azure CLI

```
sudo yum install azure-cli -y
```

- Login and sync your Azure CLI (Linux machine) to your portal.azure.com account

```
az login
```

- Now your linux machine has been fully authenticated with AZURE login.  

# Step 2 - first install docker in the local linux machine

```
 yum install epel-release -y
 yum repolist
```

## Step 2.1 - Install the required dependencies:

```
yum install yum-utils device-mapper-persistent-data lvm2  bash-completion -y
```

```
source /etc/profile.d/bash_completion.sh
```


## Step 2.2 - Add the stable Docker repository by typing:

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

## Step 2.3 - Now that we have Docker repository enabled, we can install the latest version of Docker CE (Community Edition) using yum by typing:

```
yum install docker-ce --allowerasing -y

```

## Step 2.4 - Install Docker Compose and Test the Docker-Compose Version:


- Run the below command to download the current stable release of Docker compose.

```
curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose
```

- Apply the executable permission for the binary file which we have downloaded.

```
chmod +x /usr/bin/docker-compose
```

- If the docker compose is installed on a different location For example: /usr/local/bin/ , You can copy the executable to /usr/bin directory.

- You can check the version of docker-compose using the below command.

```
docker-compose --version
```


## Step 2.5 - Once the Docker package is installed, we start the Docker daemon with:

```
systemctl start docker;systemctl status docker;systemctl enable docker
```

## Step 2.6 - To verify that the Docker service is running type:

```
systemctl status docker
```


## Step 2.7 - Enable the Docker service to be automatically started at boot time:

```
systemctl enable docker
```


## Step 2.8 - At the time of the writing of this article, the current stable version of Docker is 18.03.1, we can check our Docker version by typing:

```
docker -v
```


# Step 3: Download sample application

```
yum install git tree -y

```
- Download the sample azure application

```
git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
```

```
cd azure-voting-app-redis
```
- Build the docker image using docker compose

```
docker-compose up -d
```
- list the docker images

```
docker images
```

- list the running docker container

```
docker ps -a
```

# Step 4: Test sample application

- Test application locally

- To see the running application, enter http://localhost:8080 in a local web browser

- To see the running application, enter http://<your VM Public IP>:8080 in a local web browser

# Step 5: Deploy and use Azure Container Registry

- Create AZURE resource Group

```
az group create --name myResourceGroup --location eastus
```

- Create AZURE Container Registry Under the above Created Resource Group.


```
az acr create --resource-group myResourceGroup --name cnlacr1 --sku Basic
```

- Login to the Azure Container registry


```
az acr login --name cnlacr1
```

- List the Docker Images


```
docker images
```

- List the no of ACR in your portal.azure.com


```
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

- you will get the below output.So your AZURE Container Registry Name is below ...


```
cnlacr1.azurecr.io
```


- you need to change the docker image name towards to match your ACR repo name.Or else while you push the image will end up with error.


```
docker tag mcr.microsoft.com/azuredocs/azure-vote-front:v1 cnlacr1.azurecr.io/azure-vote-front:v1
```
- List the docker images

- Make sure you are seeing the docker image name with match to your ACR repository.

```
docker images
```
- Push your prepared custom image to your newly created ACR.

```
docker push cnlacr1.azurecr.io/azure-vote-front:v1
```

- list the ACR repository revisions

```
az acr repository list --name cnlacr1 --output table
```
- List your repository in ACR.

```
az acr repository show-tags --name cnlacr1 --repository azure-vote-front --output table
```


# Step 6: install kubectl command 

- Configure Kubernetes Software package repo.

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

- Install Kubectl

```
yum install -y kubectl
```

- To ensure all are ok the following command should work without any error

```
kubectl version --client
```

# Step 7: Deploy an Azure Kubernetes Service (AKS) cluster

- Create kubernetes Cluster with 1 worker Node

- In your learning setup,if you have project portal.azure.com test account then increase node count from 1 to 3.


```
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --generate-ssh-keys \
    --attach-acr cnlacr1
	
```
- Install AZURE AKS CLI

```
az aks install-cli
```
- Reterive the AKS cluster credentials.This will help kubectl command to run without any issues.

```
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```
- List all the nodes in Kubernetes cluster.

```
kubectl get nodes
```


# Step 8: Run applications in Azure Kubernetes Service (AKS)

- List the ACR 

```
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```
- Change your front end to your recently deployed own image repo.

```
vi azure-vote-all-in-one-redis.yaml
```


```
containers:
- name: azure-vote-front
  image: cnlacr1.azurecr.io/azure-vote-front:v1
```
- Apply the changes into AKS k8s cluster.

```
kubectl apply -f azure-vote-all-in-one-redis.yaml
```
- Monitor the change progress.

```
kubectl get service azure-vote-front --watch
```


- from the above command output get the EXTERNAL-IP and access it from browser

