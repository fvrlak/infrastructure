```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```


Running Prometheus and Grafana on Kubernetes as pods involves creating deployments and services for both Prometheus and Grafana. You can use Helm, which is a package manager for Kubernetes, to simplify this process, or you can manually create the necessary Kubernetes resources.

Here, I'll provide both methods: using Helm (which is recommended for simplicity) and manually setting up Prometheus and Grafana.

### **Method 1: Using Helm**

**Step 1: Install Helm**
If you don't have Helm installed, you can install it by following the instructions on the [Helm website](https://helm.sh/docs/intro/install/).

**Step 2: Add the Prometheus and Grafana Helm repositories**

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

**Step 3: Install Prometheus**

```sh
helm install prometheus prometheus-community/prometheus
```

This command will install Prometheus in your Kubernetes cluster.

**Step 4: Install Grafana**

```sh
helm install grafana grafana/grafana
```

This command will install Grafana in your Kubernetes cluster.

**Step 5: Access Prometheus and Grafana**

- **Prometheus**: Forward the Prometheus service to your local machine:

  ```sh
  kubectl port-forward svc/prometheus-server 9090:80
  ```

  You can then access Prometheus at `http://localhost:9090`.

- **Grafana**: Forward the Grafana service to your local machine:

  ```sh
  kubectl port-forward svc/grafana 3000:80
  ```

  You can then access Grafana at `http://localhost:3000`.

  The default Grafana login is `admin` for both the username and password. Helm should generate a new password which you can find with:

  ```sh
  kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
  ```