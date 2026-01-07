## ✅ Python Installation Guide (Use **Python 3.12 ONLY**)

- **Install Python version: 3.11**
  - Versions **above 3.11** have limited ML library support
  - **Lower versions** may cause compatibility and dependency issues

---

### 🔗 Step 1: Open Official Python Download Page
Go to:
https://www.python.org/downloads/release/python-3127/

---

### 🪟 Step 2: Choose Correct Installer (Windows 64-bit)
- Scroll to the **Version section**
- Under **Windows**, download:
  - **Windows installer (64-bit)** → *Recommended*

---

### ⚙️ Step 3: Install Python
- Run the downloaded installer
- **IMPORTANT:**  
  ✔️ Tick **“Add Python to PATH environment variables”**  
- Continue installation with default settings

---

### 🧪 Step 4: Create Virtual Environment Using Python 3.11
Make sure your virtual environment is created using **Python 3.11**, not any other version.

This ensures:
- Proper ML library support  
- Fewer dependency issues  
- Stable development environment


---

## Dockerfile

You can **copy and paste it directly** into a file named `Dockerfile`.

```dockerfile
FROM python:3.11

WORKDIR /app

COPY . /app

RUN pip install --no-cache-dir -e .

EXPOSE 5000

ENV FLASK_APP=application.py

CMD ["python" , "application.py"]
```

## Kubernetes Deployment File

You can **copy and paste it directly** into a file named `k8s-deployment.yaml`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
  labels:
    app: flask
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      labels:
        app: flask
    spec:
      securityContext:
        runAsUser: 0  # Running as root (vulnerability)
      containers:
        - name: flask-container
          image: flask-app:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 5000
          securityContext:
            privileged: true               # Privileged container (vulnerability)
            allowPrivilegeEscalation: true
            readOnlyRootFilesystem: false
            runAsNonRoot: false
          env:
            - name: SECRET_PASSWORD
              value: "admin123"            # Hardcoded secret (vulnerability)
          volumeMounts:
            - name: host-root
              mountPath: /host             # Host path mount (vulnerability)
      volumes:
        - name: host-root
          hostPath:
            path: /                        # Host path mount (vulnerability)

---
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  type: LoadBalancer
  selector:
    app: flask
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000

```

---

## GCP Cloud Setup

---

### 1. Create a VM Instance on Google Cloud

1. Go to **Compute Engine → VM Instances**
2. Click **Create Instance**

**Basic Configuration**

* **Name:** `Whatever you want to name`
* **Machine Type:**

  * Series: **E2**
  * Preset: **Standard**
  * Memory: **16 GB RAM**
* **Boot Disk:**

  * Size: **150 GB**
  * Image: **Ubuntu 24.04 LTS**
* **Networking:**

  * Enable **HTTP** and **HTTPS** traffic and **Port Forwarding** turned on

Click **Create** to launch the instance.

---

### 2. Connect to the VM

* Use the **SSH** button in the Google Cloud Console to connect to the VM directly from the browser.

---

### 3. Configure the VM Instance

#### Clone the GitHub Repository

```bash
git clone https://github.com/data-guru0/TESTING-9.git ( Whatver your Github repo link )
ls
cd TESTING-9
ls
```

You should now see the project files inside the VM.

---

### 4. Install Docker

1. Open a browser and search for **“Install Docker on Ubuntu”**
2. Open the **official Docker documentation** (`docs.docker.com`)
3. Copy and paste the **first command block** into the VM terminal
4. Copy and paste the **second command block**
5. Test the Docker installation:

```bash
docker run hello-world
```

---

### 5. Run Docker Without `sudo`

From the same Docker documentation page, scroll to **Post-installation steps for Linux** and run **all four commands** one by one.

The last command is used to verify Docker works without `sudo`.

---

### 6. Enable Docker to Start on Boot

From the section **Configure Docker to start on boot**, run:

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

---

### 7. Verify Docker Setup

```bash
systemctl status docker
docker ps
docker ps -a
```

Expected results:

* Docker service shows **active (running)**
* No running containers
* `hello-world` container appears in exited state

---

### 8. Configure Minikube Inside the VM

#### Install Minikube

1. Search for **Install Minikube**
2. Open the official website: `minikube.sigs.k8s.io`
3. Select:

   * **OS:** Linux
   * **Architecture:** x86
   * **Installation Type:** Binary

Copy and run the installation commands provided on the website.

---

#### Start the Minikube Cluster

```bash
minikube start
```

Minikube uses **Docker internally**, which is why Docker was installed first.

---


---

### 9. Verify Kubernetes & Minikube Setup

```bash
minikube status
minikubr kubectl get nodes
minikube kubectl cluster-info
docker ps
```

Expected results:

* All Minikube components are running
* A single `minikube` node is visible
* Kubernetes cluster information is accessible
* Minikube container is running in Docker

---

### 10. Install kubectl

 - Search: `Install kubectl`
  - Instead of installing manually, go to the **Snap section** (below on the same page)

  ```bash
  sudo snap install kubectl --classic
  ```

  - Verify installation:

    ```bash
    kubectl version --client
    ```

### 11. Configure GCP Firewall (If Needed)

If Jenkins does not load, create a firewall rule:

* **Name:** `allow-apps`
* **Description:** Allow all traffic (for Jenkins demo)
* **Logs:** Off
* **Network:** default
* **Direction:** Ingress
* **Action:** Allow
* **Targets:** All instances
* **Source IP ranges:** `0.0.0.0/0`
* **Allowed protocols and ports:** All

---

## Enable Useful Minikube Addons

Enable the following Minikube addons to enhance cluster functionality:

```bash
minikube addons enable dashboard
minikube addons enable metrics-server
minikube addons enable ingress
```


## Install Required Dependencies

Update the system and install essential packages:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget apt-transport-https ca-certificates gnupg lsb-release python3-pip git
```

## Get Minikube IP

Retrieve the Minikube cluster IP and store it for later use:

```bash
minikube ip
```

## Install & Run kube-hunter Using Docker

### Log in to DockerHub

> **Note:** If you encounter an error while running the kube-hunter container, make sure you are logged in to DockerHub first.  
> Create a **Personal Access Token (PAT)** from DockerHub and use it as the password during login.

```bash
docker login -u dataguru97
````
* Use your username here...
When prompted:

* **Username:** `dataguru97`
* **Password:** Paste your **DockerHub Personal Access Token (PAT)**

After successful authentication, DockerHub access will be enabled inside your VM.


### Generate Output on terminal

```bash
docker run -it --network minikube --rm aquasec/kube-hunter --remote 192.168.49.2 --report plain
```

- Make sure to use your minikube ip here....

### Send output to a JSON File

```bash
docker run -it --network minikube --rm aquasec/kube-hunter --remote 192.168.49.2 --report json > kube-hunter-report.json
```

- Make sure to use your minikube ip here....

##  Build and Deploy Your App on VM

### Point Docker to Minikube
```bash
eval $(minikube docker-env)
````


### Build Docker Image

```bash
docker build -t flask-app:latest .
```


### Deploy Application to Kubernetes

```bash
kubectl apply -f k8s-deployment.yaml
```


### Check Pod Status

```bash
kubectl get pods
```

> You will see the pods running.


### Expose the App (Port Forwarding)

```bash
kubectl port-forward svc/flask-service 5000:80 --address 0.0.0.0
```

- Make sure to give correct service name.


### ✅ Access the Application

* Copy the **External IP**
* Open browser and visit:
- http://EXTERNAL-IP:5000

🎉 Your application is now live..
---


