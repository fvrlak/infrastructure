To install ArgoCD using Helm, follow these steps:

### Step 1: Add the ArgoCD Helm Repository
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

### Step 2: Create a Namespace for ArgoCD
```bash
kubectl create namespace argocd
```

### Step 3: Install ArgoCD using Helm
```bash
helm install argocd argo/argo-cd --namespace argocd
```

### Step 4: Access the ArgoCD UI
To access the ArgoCD UI, you need to expose the `argocd-server` service.

#### Option 1: Port Forwarding (for local access)
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
You can now access the ArgoCD UI at `https://localhost:8080`.


### Step 5: Retrieve the Initial Admin Password
```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
```

Use `admin` as the username and the decoded password to log in to the ArgoCD UI.

### Step 6: Connect Argo CD with repository
Generate ssh keys and put .pub into github repo.

Navigate to `localhost:8080` and use `admin` and as a password the retrieval of Step 5

- Settings -> Connect Repo -> SSH Connection Method
  - Name : InfiniteCanvas 
  - Project: default
  - Repository URL : SSH or repository
  - SSH private key data: `cat ~/.ssh/id_ed25519`
    
### Step 7: Register Cluster with Argo CD
To register a Kubernetes cluster with ArgoCD, you need to use the ArgoCD CLI (`argocd`). Here's a step-by-step guide:

### 1. **Install ArgoCD CLI**
   If you haven't already installed the ArgoCD CLI, you can do so by following these steps:

   - On macOS using Homebrew:
     ```bash
     brew install argocd
     ```

   - On Linux:
     ```bash
     sudo curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
     sudo chmod +x /usr/local/bin/argocd
     ```

   - On Windows, you can download the executable from the [ArgoCD GitHub Releases](https://github.com/argoproj/argo-cd/releases).

### 2. **Log in to ArgoCD**
   Use the ArgoCD CLI to log in to your ArgoCD server.

   ```bash
   argocd login <ARGOCD_SERVER>
   ```

   Replace `<ARGOCD_SERVER>` with the hostname or IP address of your ArgoCD server (e.g., `argocd.example.com`).

   You’ll be prompted for a username and password:
   - **Username**: `admin` (default, unless changed)
   - **Password**: The password you retrieved when installing ArgoCD

   Example:
   ```bash
   argocd login argocd.example.com
   ```

### 3. **Register Your Kubernetes Cluster**
   After logging in, you can register your Kubernetes cluster with ArgoCD.

   ```bash
   argocd cluster add <CONTEXT_NAME>
   ```

   Replace `<CONTEXT_NAME>` with the name of the Kubernetes context you want to register. You can find the context names by running `kubectl config get-contexts`.

   Example:
   ```bash
   argocd cluster add minikube
   ```

   During this process, you’ll be asked to grant ArgoCD the necessary permissions on your cluster. The `argocd` CLI will create a service account in the `kube-system` namespace and configure ArgoCD to use it.

### 4. **Verify the Cluster Registration**
   You can verify that the cluster was successfully added by running:

   ```bash
   argocd cluster list
   ```

   This will show you a list of all clusters that are registered with ArgoCD.

### 5. **Configure ArgoCD Application to Use the Registered Cluster**
   - When you create or update an application in ArgoCD, you’ll be able to select this newly registered cluster as the target for deployment.
   - In the ArgoCD UI, when setting up an application, you can now choose the registered cluster from the "Cluster" drop-down list.

### Summary of the Workflow

1. **Install ArgoCD CLI**: Ensure you have the `argocd` CLI installed.
2. **Log in to ArgoCD**: Use the CLI to log in to your ArgoCD server.
3. **Register the Cluster**: Use `argocd cluster add <CONTEXT_NAME>` to register your Kubernetes cluster with ArgoCD.
4. **Use the Cluster in ArgoCD**: The cluster will now be available in the ArgoCD UI for deploying applications.

This registration process allows ArgoCD to deploy applications to your Kubernetes cluster and manage them using a GitOps workflow.





### Step 6: Optional - Custom Configuration
If you need to customize the installation, you can create a `values.yaml` file with your desired settings and install ArgoCD using that file:
```bash
helm install argocd argo/argo-cd --namespace argocd -f values.yaml
```

This will set up ArgoCD in your Kubernetes cluster, allowing you to start managing your applications with GitOps.