# Prometheus & Grafana

## Create Namespace

```shell
k create ns monitoring
```

## Create ClusterRole

```shell
k apply -f kubernetes/manifests/clusterRole.yaml
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
```

## Externalize Prometheus Configuration

```shell
k apply -f kubernetes/manifests/config-map.yaml
configmap/prometheus-server-conf created
```

## Create the Prometheus Deployment

```shell
k apply -f kubernetes/manifests/prometheus-deployment.yaml
deployment.apps/prometheus-deployment created
```

## Validate the Deployment

```shell
kubectl -n monitoring get deployments
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
prometheus-deployment   1/1     1            1           60s
```

## Expose Prometheus as LoadBalancer

```shell
k apply -f kubernetes/manifests/prometheus-service.yaml
service/prometheus-service created
```

## How to delete Prometheus

### Delete the Service

```shell
k delete -f kubernetes/manifests/prometheus-service.yaml
service "prometheus-service" deleted
```

### Delete the Deployment

```shell
k delete -f kubernetes/manifests/prometheus-deployment.yaml
deployment.apps "prometheus-deployment" deleted
```

### Delete Config Map Configuration

```shell
k delete -f kubernetes/manifests/config-map.yaml
configmap "prometheus-server-conf" deleted
```

### Delete the ClusterRole

```shell
k delete -f kubernetes/manifests/clusterRole.yaml
clusterrole.rbac.authorization.k8s.io "prometheus" deleted
clusterrolebinding.rbac.authorization.k8s.io "prometheus" deleted
```

### Delete the Namespace

```shell
k delete ns monitoring
namespace "monitoring" deleted
```

## Installing Prometheus & Grafana with Helm

### Add the Helm Repository

```shell
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
"prometheus-community" already exists with the same configuration, skipping
```

### Update the Repository

```shell
helm repo update
```

### Search for Prometheus Repository

```shell
helm search repo prometheus
```

### Install Prometheus

```shell
helm install prometheus prometheus-community/prometheus
```

### Install Prometheus LoadBalancer

```shell
k apply -f kubernetes/manifests/helm-prometheus-service.yaml
service/prometheus-sever-ext created
```

## Install Grafana

### Install the Grafana Helm Repo

```shell
helm repo add grafana https://grafana.github.io/helm-charts
"grafana" already exists with the same configuration, skipping
```

### Install Grafana

```shell
helm install grafana grafana/grafana
```

### Create the Grafana Service

```shell
k apply -f kubernetes/manifests/helm-grafana-service.yaml
service/grafana-sever-ext created
```

### Validate the Grafana External Service

```shell
k get svc grafana-ext
NAME          TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
grafana-ext   LoadBalancer   10.96.80.182   localhost     9091:31810/TCP   38s
```

### Get the Grafana Password

The default username is `admin` and you can get the password with the following command:

```shell
k get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

## Install Kube Prometheus Stack

### Get the Helm Chart Values

```shell
helm show values prometheus-community/kube-prometheus-stack > kubernetes/manifests/values.yaml
```

```shell
helm install prometheus prometheus-community/kube-prometheus-stack --set nodeExporter.enable=false
```

### Patch the prometheus-prometheus-node-exporter

```shell
kubectl patch ds prometheus-prometheus-node-exporter --type "json" -p '[{"op": "remove", "path" : "/spec/template/spec/containers/0/volumeMounts/2/mountPropagation"}]'
daemonset.apps/prometheus-prometheus-node-exporter patched
```

```shell
k expose deploy prometheus-grafana --type=LoadBalancer --port=9090 --target-port=80 --name=prometheus-grafana-ext -o yaml --dry-run=client > kubernetes/manifests/helm-prometheus-grafana.yaml
```

```shell
k apply -f kubernetes/manifests/helm-prometheus-grafana.yaml
service/prometheus-grafana-ext created
```

```shell
k get secret --namespace default prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
prom-operator
```

## References

- [Setup Prometheus Monitoring on Kubernetes](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/)