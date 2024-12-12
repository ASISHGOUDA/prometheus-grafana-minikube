# Prometheus and Grafana Installation Using Helm Charts

This guide explains how to install Prometheus and Grafana in a Kubernetes cluster using Helm charts.

## Prerequisites

Before you begin, ensure the following:

1. A running Kubernetes cluster.
2. `kubectl` CLI installed and configured.
3. Helm CLI installed (version 3 or later).

## Step 1: Add Helm Repositories

Add the Prometheus and Grafana Helm chart repositories:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

## Step 2: Install Prometheus

1. Create a dedicated namespace for monitoring components:
   ```bash
   kubectl create namespace monitoring
   ```

2. Install Prometheus using the `kube-prometheus-stack` Helm chart:
   ```bash
   helm install prometheus prometheus-community/kube-prometheus-stack \
     --namespace monitoring
   ```

3. Verify the Prometheus installation:
   ```bash
   kubectl get pods -n monitoring
   ```

## Step 3: Install Grafana

1. Install Grafana in the same namespace:
   ```bash
   helm install grafana grafana/grafana \
     --namespace monitoring
   ```

2. Verify the Grafana installation:
   ```bash
   kubectl get pods -n monitoring
   ```

3. Retrieve the Grafana admin password:
   ```bash
   kubectl get secret grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode
   ```
   The default username is `admin`.

## Step 4: Access Prometheus and Grafana

1. **Prometheus**:
   - Forward the Prometheus service port:
     ```bash
     kubectl port-forward svc/prometheus-kube-prometheus-prometheus -n monitoring 9090:9090
     ```
   - Open your browser and navigate to `http://localhost:9090`.

2. **Grafana**:
   - Forward the Grafana service port:
     ```bash
     kubectl port-forward svc/grafana -n monitoring 3000:80
     ```
   - Open your browser and navigate to `http://localhost:3000`.

## Step 5: Configure Grafana

1. Login to Grafana using the admin credentials retrieved earlier.
2. Add Prometheus as a data source:
   - Navigate to **Configuration → Data Sources → Add Data Source**.
   - Select **Prometheus**.
   - Set the URL to `http://prometheus-kube-prometheus-prometheus.monitoring.svc.cluster.local:9090`.
   - Click **Save & Test**.

3. Import Prebuilt Dashboards:
   - Go to **Create → Import**.
   - Enter the Dashboard ID from [Grafana's Dashboard Library](https://grafana.com/grafana/dashboards/).
   - Example: Use Dashboard ID `315` for Kubernetes cluster monitoring.

## Step 6: Uninstall (Optional)

To uninstall Prometheus and Grafana:

1. Uninstall Prometheus:
   ```bash
   helm uninstall prometheus -n monitoring
   ```

2. Uninstall Grafana:
   ```bash
   helm uninstall grafana -n monitoring
   ```

3. Delete the monitoring namespace:
   ```bash
   kubectl delete namespace monitoring
   ```

## Notes

- Ensure sufficient resources are allocated to your Kubernetes cluster for running Prometheus and Grafana.
- Use PersistentVolumes if you need long-term data storage for Prometheus or Grafana configurations.
- Secure Grafana with Single Sign-On (SSO) or by enabling HTTPS for production use.
