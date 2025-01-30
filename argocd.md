#  Automated Deployment of Real-Time Chat Application to Kubernetes with ArgoCD

Welcome to the official guide for deploying a **Full-Stack Chat Application** on kuberentes woth help of argocd. Whether you're a student, a professional, or someone exploring the world of Kubernetes and Docker, this guide is designed to help you deploy the app with ease.

In this tutorial, you will learn how to:

1. Set up a local Kubernetes environment using **Kind** and install some other prerequisites like kubectl ,docker etc.
2. Deploy **Argocd** on kind cluster and expose it .
3. Deploy a **Real-Time Chat Application** (Frontend, Backend, MongoDB) on **Kubernetes** using **Argocd** and access application.
4.test and validate the deployments 

## Tools required :+1: 

* Kind (Kubernetes in Docker): A tool for running Kubernetes clusters in Docker containers.
* Kubectl: The Kubernetes command-line tool to interact with Kubernetes clusters.
* Docker: Essential for building and running containers.
* ArgoCD: A GitOps continuous delivery tool for Kubernetes.
* Git: For managing ,tracking and storing source code in a repository.
* Helm (optional): If you choose to use Helm for managing Kubernetes applications, it simplifies deployment with charts.


---


## 📋1. Prerequisites

Before we start the deployment, ensure that you have the following tools installed and set up on your machine:

### **1. Docker** 
Install Docker

windows -
in windows you  can install directly docker desktop from this website 
https://docs.docker.com/desktop/setup/install/windows-install/

linux -

```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```
sudo docker run hello-world
```


### **2. Kind (Kubernetes in Docker)**  
Kind is a tool that lets you run Kubernetes clusters in Docker containers. It’s lightweight, easy to use, and perfect for local development.

**For Windows** (PowerShell):
```bash
curl.exe -Lo kind-windows-amd64.exe https://kind.sigs.k8s.io/dl/v0.25.0/kind-windows-amd64
Move-Item .\kind-windows-amd64.exe c:\some-dir-in-your-PATH\kind.exe
```

**For Windows** (WSL - Windows Subsystem for Linux):
```bash
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.25.0/kind-linux-amd64

# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.25.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

**For Linux**:
```bash
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.25.0/kind-linux-amd64

# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.25.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

---

### **3. Kubectl (Kubernetes Command Line Tool)**  
Kubectl is the tool we’ll use to manage Kubernetes clusters. You’ll use it to interact with your local Kubernetes cluster and deploy resources.

**For x86_64 Architecture:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

**For ARM64 Architecture:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/arm64/kubectl"
```

**Validate the downloaded binary:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```

**Install kubectl:**
```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
# OR if there’s an issue with your root permissions:
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
```

**Verify kubectl installation:**
```bash
kubectl version --client --output=yaml
```

---


### 4.Cloning the Project

With the prerequisites set up, let’s grab the code for the chat application. Run the following commands:

```bash
git clone https://github.com/deepak4566/Realtime-chat-app.git
```
```bash
cd Realtime-chat-app/k8s
```


This will:
- **Clone** the project repository from GitHub.
- Navigate into the `k8s` folder, which contains the Kubernetes configuration files for the deployment.

---

## 🚢 2. Installing ArgoCD on a Kind Cluster and Exposing it on port 9090.

Before deploying resources into Kubernetes, we need to install and deploy ArgoCD to implement GitOps.

### 1. Install ArgoCD

Run the following commands to create the ArgoCD namespace and install ArgoCD:

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```



### 2. Access the Argo CD UI**
1. Get the Argo CD server URL:
  
   - Access using port-forwarding:
     ```sh
     kubectl port-forward svc/argocd-server -n argocd 9090:443
     ```
     Then, access **https://localhost:9090** in your browser.

2. **Login to Argo CD**:
   - Username: `admin`
   - Password:
     ```sh
     kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
     ```
     or
     
     ```sh
     kubectl edit secret argocd-initial-admin-secret -n argocd 
     ```
     copy the secret and paste in this base64 decoder https://www.base64decode.org/   
     



---


## 3. **Deploying an Application Using Argo CD UI**

### **Step 1: Access the Argo CD UI**
1. Get the Argo CD server URL:
   - If exposed via a Clusterip/Nodeport/Loadbalancer:
     ```sh
     kubectl get svc -n argocd
     ```
   - If using port-forwarding:
     ```sh
     kubectl port-forward svc/argocd-server -n argocd 9090:443
     ```
     Then, access **https://localhost:9090** in your browser.
     ![a1](https://hackmd.io/_uploads/BJsn6RuuJl.png)


2. **Login to Argo CD**:
   - Username: `admin`
   - Password:
     ```sh
     kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
     ```
     or
     
     ```sh
     kubectl edit secret argocd-initial-admin-secret -n argocd 
     ```
     copy the secret and paste in this base64 decoder https://www.base64decode.org/   
     

### **Step 2: Connect Argo CD to Your Git Repository**

1. In the **Argo CD UI**, go to **"Repositories"**.
2. Click **"Connect Repo"**.
3. Enter the **Git repository URL** (e.g., `https://github.com/deepak4566/Realtime-chat-app.git`).
4. Choose **authentication method** (HTTPS, SSH, or username/password).
5. Click **"Connect"** to save.
6. **Note**: If the GitHub repository is private, provide a GitHub personal access token or authenticate using a username and password.


![a8](https://hackmd.io/_uploads/HkeMxa0__yl.png)


---

### **Step 3: Create a New Argo CD Application**
1. Navigate to **"Applications"** → Click **"Create Application"**.
![a3](https://hackmd.io/_uploads/r1Aw0Cduyx.png)
2. Fill in the details:
   - **Application Name**: `chatapp`
   - **Project**: `default`
   - **Sync Policy**: Manual / Automatic
   - **Repository URL**: Select the connected Git repo
   - **Path**: (e.g., `k8s`)
   - **Cluster**: Select your Kubernetes cluster
   - **Namespace**: chat-app
 ![a4](https://hackmd.io/_uploads/rJacA0_O1g.png)
 ![a5](https://hackmd.io/_uploads/H13sAAu_Jl.png)

  
   
3. Click **"Create"**.
![a6](https://hackmd.io/_uploads/rJ1ACAdOyx.png)



### **Step 4: Verify Deployment**
![a7](https://hackmd.io/_uploads/rJYe1ktOyx.png)

1. In the **Argo CD UI**, check the status of the application:
   - ✅ Green: Running successfully
   - ⚠️ Yellow: Pending changes
   - ❌ Red: Errors in deployment
2. To check Kubernetes resources, go to **"Application Details" → "Resources"**.

---

### **Step 5: Configure Auto-Sync (Optional)**
- If you want Argo CD to automatically deploy changes from Git:
  1. Open the application.
  2. Click **"Edit"**.
  3. Enable **"Auto-Sync"** and **"Self-Heal"**.
  4. Save the changes.


### **Step 6: Verification** 

Once the app is deployed, it's crucial to verify that everything is running smoothly.

## Check all resources
```
kubectl get all -n chat-app
```
![a9](https://hackmd.io/_uploads/SkjEUkYdyg.png)

## Check pod logs if needed
```
kubectl logs -f -l app=frontend -n chat-app
kubectl logs -f -l app=backend -n chat-app
kubectl logs -f -l app=mongodb -n chat-app
```

---
### **Step 7: Access the application**

open new terminal

## port-forward the service
```
kubectl port-forward -n chat-app service/frontend 8080:80
```
![a10](https://hackmd.io/_uploads/rJ0pI1tOkx.png)

## Accessing the Application
The application is exposed through port forwarding : http://localhost:8080

![a11](https://hackmd.io/_uploads/rJ8wPyKOkg.png)

### **1: Access the application**


## Accessing the Application
The application is exposed through port forwarding : http://localhost:8080

### **2: Signup and create account for user 1**
![a13](https://hackmd.io/_uploads/BkYsGgt_ke.png)


### **3: Signup and create account for user 2 in different browser**
Signup and create account for user 2 in different browser
now we  can start chatting with user1 by using same url

user2 initiating chat-
![a14](https://hackmd.io/_uploads/Hy6AfgFdyx.png)

user1 replying to messages of user2-
![a15](https://hackmd.io/_uploads/r11mXxtOye.png)




---


Now, every time you push updates to your Git repo, Argo CD will automatically sync and deploy them. 🚀


## 4. **Test and validate deployemnt**

### validation -> database validation

Now as we created two users from ui and also done some messaging let us check whether those data is presisted in these pv(persistent voulme)

### **1: Exec into mongodb pod**

To **log into MongoDB** from the container's shell, follow these steps:

1. **Ensure you're in the container's shell (not the MongoDB shell):**
   Execute this command from your local machine:
   ```sh
   kubectl exec -it mongodb-deployment-688778787d-tnpnw -c chatapp-mongodb -- /bin/bash
   ```
   ![a16](https://hackmd.io/_uploads/HynEBgtuye.png)

2. **Once inside the container shell**, run this exact command:


   - For MongoDB 5.0 and later (using `mongosh`):
     ```sh
     mongosh "mongodb://mongoadmin:secret@localhost:27017/admin"
     ```

   - For MongoDB 4.x and earlier (using `mongo`):
     ```sh
     mongo "mongodb://mongoadmin:secret@localhost:27017/admin"
     ```
     
To use list databases and use correct database apply these commands

```
show dbs
```
```
use dbname
```
     
![a17](https://hackmd.io/_uploads/H1gnLeYukx.png)


3. **Once inside the database**, to check created messages ,users respectively :

```
db.messages.find()
```

![a18](https://hackmd.io/_uploads/ryaOFxKuJx.png)

```
db.users.find()
```
![a19](https://hackmd.io/_uploads/HJ9stxYOJx.png)


## Cleanup

When you're done, you can clean up using:
```bash
# Delete all resources in namespace
kubectl delete namespace chat-app

# Delete the kind cluster
kind delete cluster --name chat-app-cluster
```

---



## 🎉 Conclusion

Congratulations! You’ve successfully deployed the **Full-Stack Real time Chat Application** using **Argocd in Kubernetes** . Whether you're using Kubernetes for a more robust, scalable solution or your chat app is now running!
