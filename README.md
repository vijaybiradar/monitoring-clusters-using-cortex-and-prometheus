# monitoring-clusters-using-cortex-prometheus-and-grafana
Add Helm repositories:
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add cortex-helm https://cortexproject.github.io/cortex-helm-chart
helm repo update
```

Install Cortex:

Create a values file (let's name it cortex-values.yaml) with the Cortex configuration you provided earlier. Here's an example of how it might look:

vi cortex-values.yaml
```
metaMonitoring:
  serviceMonitor:
    enabled: true
    labels:
      release: kube-prometheus-stack
config:
  blocks_storage:
    backend: s3
    s3:
      bucket_name: cortex-prod-north-virginia
      endpoint: s3.us-east-1.amazonaws.com
      region: us-east-1
      access_key_id: "your-access-key-id"
      secret_access_key: "your-secret-access-key"
    tsdb:
      dir: /data/tsdb
    bucket_store:
      sync_dir: /data/tsdb
      bucket_index:
        enabled: true
  ruler_storage:
    s3:
      bucket_name: cortex-prod-north-virginia
      endpoint: s3.us-east-1.amazonaws.com
      region: us-east-1
      access_key_id: "your-access-key-id"
      secret_access_key: "your-secret-access-key"
  alertmanager_storage:
    s3:
      bucket_name: cortex-prod-north-virginia
      endpoint: s3.us-east-1.amazonaws.com
      region: us-east-1
      access_key_id: "your-access-key-id"
      secret_access_key: "your-secret-access-key"
alertmanager:
  serviceMonitor:
    enabled: false
    additionalLabels:
      release: kube-prometheus-stack

distributor:
  serviceMonitor:
    enabled: true
    additionalLabels:
      release: kube-prometheus-stack

ingester:
  serviceMonitor:
    enabled: true
    additionalLabels:
      release: kube-prometheus-stack

ruler:
  serviceMonitor:
    enabled: false
    additionalLabels:
      release: kube-prometheus-stack

querier:
  serviceMonitor:
    enabled: true
    additionalLabels:
      release: kube-prometheus-stack

query_frontend:
  serviceMonitor:
    enabled: true
    additionalLabels:
      release: kube-prometheus-stack

nginx:
  service:
  serviceMonitor:
    enabled: true
    additionalLabels:
      release: kube-prometheus-stack

store_gateway:
  serviceMonitor:
    enabled: true
    additionalLabels:
      release: kube-prometheus-stack

compactor:
  serviceMonitor:
    enabled: true
    additionalLabels:
      release: kube-prometheus-stack
```

Replace "your-access-key-id" and "your-secret-access-key" with your actual AWS access key ID and secret access key.

install Cortex using Helm:

```
helm install cortex cortex-helm/cortex -f cortex-values.yaml --namespace cortex
```
Install kube-prometheus-stack:
Create a values file (let's name it kube-prometheus-stack-values.yaml) for kube-prometheus-stack:

vi kube-prometheus-stack-values.yaml
```
prometheus:
  prometheusSpec:
    remoteWrite:
      - url: http://cortex.cortex:80/api/v1/push
    externalLabels:
       environment: cortex-prod
```
install kube-prometheus-stack using Helm:

```
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -f kube-prometheus-stack-values.yaml --namespace monitoring
```
This setup will deploy Cortex with S3 storage backend and kube-prometheus-stack configured to send metrics to Cortex. Adjust configurations as per your specific environment and requirements.


Access Grafana Dashboard

Port Forward Grafana Service
```
kubectl port-forward svc/prom-grafana 3000:80 -n monitoring
```

Access Grafana Dashboard Open a web browser and go to localhost:3000.

![image](https://github.com/vijaybiradar/monitoring-clusters-using-cortex-and-prometheus/assets/38376802/5695ea51-ca03-4cbd-be42-ed6d5bb265da)


Login with the following credentials: Username: admin Password: (retrieve password using the following command)
```
kubectl get secret prom-grafana -o jsonpath="{.data.admin-password}" -n monitoring | base64 --decode ; echo
```
Username: admin Password: prom-operator
