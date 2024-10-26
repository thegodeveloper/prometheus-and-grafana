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

## References

- [Setup Prometheus Monitoring on Kubernetes](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/)