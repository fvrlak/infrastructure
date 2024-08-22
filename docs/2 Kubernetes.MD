To set up a Kubernetes cluster on Ubuntu 22.04.3 LTS, follow these steps:

## 1. **Update System**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

## 2. **Install Docker**

### Step 1: Download Docker Desktop
1. **Download Docker** 
   ```bash
   https://www.docker.com/products/docker-desktop
   ```
2. **Complete the Installation**:
   - Once the installation is complete, Docker Desktop will start automatically. You should see the Docker icon in your system tray (bottom-right corner of your screen).

### Step 2: Enable WSL 2 Integration in Docker Desktop

1. **Open Docker Desktop**:
   - Click on the Docker icon in the system tray to open Docker Desktop.

2. **Access Settings**:
   - In Docker Desktop, click on the gear icon (⚙️) to open **Settings**.

3. **Enable WSL 2 Integration**:
   - Go to **Resources** > **WSL Integration**.
   - You will see a list of WSL 2 distributions installed on your system (e.g., Ubuntu-22.04).
   - **Toggle the switch** next to the Ubuntu distribution (or any other WSL 2 distro) to enable Docker integration with it.

4. **Apply and Restart**:
   - After enabling WSL 2 integration, click **Apply & Restart** to apply the changes.

### Step 3: Test  Docker in Your WSL 2 Ubuntu Environment

1. **Run Docker Commands**:
   - You can now run Docker commands directly in your WSL 2 terminal. For example:
     ```bash
     docker --version
     docker run hello-world
     ```
   - These commands will interact with the Docker daemon running on Windows through the WSL 2 integration.

### Summary:
- **Docker Desktop**: Provides a seamless Docker experience on Windows by managing the Docker daemon and integrating it with WSL 2.
- **WSL 2 Integration**: Allows you to use Docker commands within your WSL 2 Ubuntu environment, making it feel like Docker is running natively on Linux.
- **No Systemd Required**: This setup bypasses the need for `systemd`, making it ideal for WSL 2.

This setup gives you a powerful, Linux-like development environment on Windows, fully integrated with Docker.


## 3. **Install Kubectl**

1. **Download the Latest Release of `kubectl`**:
   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   ```

2. **Make the `kubectl` Binary Executable**:
   ```bash
   chmod +x kubectl
   ```

3. **Move the Binary to a Directory in Your PATH**:
   ```bash
   sudo mv kubectl /usr/local/bin/
   ```

4. **Test to Ensure It’s Working**:
   ```bash
   kubectl version --client
   ```

To install Minikube on your system, follow these steps:

## 4. **Minikube**

First, ensure you have the required dependencies installed:

- **Virtualization**: Minikube can run with different drivers, such as Docker, KVM, VirtualBox, or Hyper-V. If you have Docker installed (as you mentioned earlier), you can use it as the driver.
- **Curl**: For downloading Minikube.

```bash
sudo apt-get update
sudo apt-get install -y curl apt-transport-https
```

### 1. **Install**

Download the latest Minikube binary and install it:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### 2. **Start Minikube**

Start Minikube using the Docker driver (assuming Docker is installed and running):

```bash
minikube start --driver=docker
```

This command will start a Minikube cluster using Docker as the driver. If you want to use a different driver, specify it with the `--driver` flag (e.g., `virtualbox`, `kvm2`, etc.).

### 3. **Verify Installation**

To verify that Minikube is installed and running:

```bash
minikube status
```

You should see the status of the Minikube cluster, which should indicate that it is running.

### 4. **Use Minikube with Kubectl**

Once Minikube is running, you can use `kubectl` to interact with the cluster:

```bash
kubectl get nodes
```

This command should show a single node (your Minikube instance) as ready.

### 5. **Minikube Dashboard (Optional)**

You can also launch the Minikube dashboard, which provides a graphical interface to manage your cluster:

```bash
minikube dashboard
```

## 6. **Run images of apps in Kubernetes:**
- Build image of the target app. The app needs to be created in minikube   environment.
  - Since I want to use metrics `server.js` i need to install `concurrently` package and add it to `package.json`.
  - Build `Dockerfile`
    ```bash
    # Use the official Node.js 20 image as a base image
    FROM node:20-alpine

    # Set the working directory inside the container
    WORKDIR /usr/src/app

    # Copy the package.json and package-lock.json to install dependencies
    COPY package*.json ./

    # Install dependencies
    RUN npm install

    # Copy the rest of the application code to the container
    COPY . .

    # Build the Next.js application
    RUN npm run build

    # Expose the ports that your app and metrics server will run on
    EXPOSE 4000 3001

    # Use 'concurrently' to run both Next.js and the metrics server
    RUN npm install concurrently --save

    CMD ["npx", "concurrently", "\"PORT=4000 npm run start\"", "\"node server.    js\""] 
    ```

  - To point terminal’s Docker client to Minikube’s Docker daemon
    ```bash
    eval $(minikube -p minikube docker-env)
    docker build -t infinite-canvas .
    ```   
  - Create a `deployment.yaml` file to define your Kubernetes deployment:
  ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: infinite-canvas
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: infinite-canvas
        template:
          metadata:
            labels:
              app: infinite-canvas
          spec:
            containers:
            - name: infinite-canvas
              image: infinite-canvas:latest
              imagePullPolicy: Never # important, else error image pull
              ports:
              - containerPort: 4000 # Next.js port
              - containerPort: 3001 # Metrics server port
  ```

  - Create a `service.yaml` file to define the application service:
  ```yaml
      apiVersion: v1
      kind: Service
      metadata:
        name: infinite-canvas-service
      spec:
        type: NodePort
        selector:
          app: infinite-canvas
        ports:
          - name: nextjs-port
            protocol: TCP
            port: 4000
            targetPort: 4000
            nodePort: 30000
          - name: metrics-port
            protocol: TCP
            port: 3001
            targetPort: 3001
            nodePort: 30001
  ```
  - Acces service:
  ```bash
  minikube service infinite-canvas-service --url
  ```
