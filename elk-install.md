# Elasticsearch, Logs and Kibana in Kubernetes

## Create a Namespace

```shell
k create ns elk
namespace/elk created
```

## Add the Elastic Helm Repository

```shell
helm repo add elastic https://helm.elastic.co
"elastic" has been added to your repositories
```

## Install Elasticsearch

### Install Elasticsearch in Kubernetes

```shell
helm install elasticsearch elastic/elasticsearch --version 8.5.1 -n elk --set replicas=1
NAME: elasticsearch
LAST DEPLOYED: Sun Oct 27 11:31:28 2024
NAMESPACE: elk
STATUS: deployed
REVISION: 1
NOTES:
1. Watch all cluster members come up.
  $ kubectl get pods --namespace=elk -l app=elasticsearch-master -w
2. Retrieve elastic user's password.
  $ kubectl get secrets --namespace=elk elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
3. Test cluster health using Helm test.
  $ helm --namespace=elk test elasticsearch
```

### Validate the Elasticsearch Installation

```shell
helm list -n elk
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
elasticsearch   elk             1               2024-10-27 11:31:28.264538 -0500 -05    deployed        elasticsearch-8.5.1     8.5.1
```

### Validate the Elasticsearch Resources Deployed

The resources should take some time to be deployed.

```shell
k -n elk get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/elasticsearch-master-0   1/1     Running   0          66s

NAME                                    TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)             AGE
service/elasticsearch-master            ClusterIP   10.98.77.46   <none>        9200/TCP,9300/TCP   66s
service/elasticsearch-master-headless   ClusterIP   None          <none>        9200/TCP,9300/TCP   66s

NAME                                    READY   AGE
statefulset.apps/elasticsearch-master   1/1     66s
```

## Install Kibana

### Install Kibana in Kubernetes

```shell
helm install kibana elastic/kibana --version 8.5.1 -n elk
NAME: kibana
LAST DEPLOYED: Sun Oct 27 11:38:18 2024 ClusterIP   10.110.112.142   <none>        9200/TCP,9300/TCP   2m5s
NAMESPACE: elkcsearch-master-headless   ClusterIP   None             <none>        9200/TCP,9300/TCP   2m5s
STATUS: deployed
REVISION: 1                             READY   AGE
TEST SUITE: None/elasticsearch-master   0/3     2m5s
NOTES:
1. Watch all containers come up.rometheus    main +1 !1 ··································································································· 11:26:59  ─╮
  $ kubectl get pods --namespace=elk -l release=kibana -w                                                                                                                    ─╯
2. Retrieve the elastic user's password.
  $ kubectl get secrets --namespace=elk elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
3. Retrieve the kibana service account token.
  $ kubectl get secrets --namespace=elk kibana-kibana-es-token -ojsonpath='{.data.token}' | base64 -d
```

### Validate the Kibana Helm Installation

```shell
helm -n elk list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
elasticsearch   elk             1               2024-10-27 11:31:28.264538 -0500 -05    deployed        elasticsearch-8.5.1     8.5.1      
kibana          elk             1               2024-10-27 11:38:18.766756 -0500 -05    deployed        kibana-8.5.1            8.5.1
```

### Validate the Kibana Deployment

```shell
k -n elk get deployment kibana-kibana
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
kibana-kibana   1/1     1            1           9m20s
```

### Validate the Kibana Pods

```shell
k -n elk get pods -l app=kibana
NAME                            READY   STATUS    RESTARTS   AGE
kibana-kibana-555ddb75f-97rqd   1/1     Running   0          10m
```

### Validate the Kibana Service

```shell
k -n elk get svc kibana-kibana
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kibana-kibana   ClusterIP   10.105.149.75   <none>        5601/TCP   11m
```

### Expose Kibana Service

```shell
k -n elk expose deploy kibana-kibana --name=kibana-ext --port=5601 --target-port=5601 --type=LoadBalancer -o yaml --dry-run=client > kubernetes/manifests/helm-kibana-service-ext.yaml
```

### Apply the Kibana External Service

```shell
k apply -f kubernetes/manifests/helm-kibana-service-ext.yaml
service/kibana-ext created
```

### Validate the Kibana External Service

```shell
k -n elk get svc kibana-ext
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kibana-ext   LoadBalancer   10.111.174.77   localhost     5601:30666/TCP   45s
```

### Get the Kibana Username

```shell
kubectl get secrets --namespace=elk elasticsearch-master-credentials -ojsonpath='{.data.username}' | base64 -d
elastic
```

### Get the Kibana Password

```shell
kubectl get secrets --namespace=elk elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
```

### Get the Kibana Service Account Token

```shell
kubectl get secrets --namespace=elk kibana-kibana-es-token -ojsonpath='{.data.token}' | base64 -d
```

### Install The Kubernetes Agent

Change the `elastic` password on line 962.

```shell
k apply -f kubernetes/manifests/elastic-agent-standalone-kubernetes.yaml
configmap/agent-node-datastreams created
daemonset.apps/elastic-agent created
clusterrolebinding.rbac.authorization.k8s.io/elastic-agent created
rolebinding.rbac.authorization.k8s.io/elastic-agent created
rolebinding.rbac.authorization.k8s.io/elastic-agent-kubeadm-config created
clusterrole.rbac.authorization.k8s.io/elastic-agent created
role.rbac.authorization.k8s.io/elastic-agent created
role.rbac.authorization.k8s.io/elastic-agent-kubeadm-config created
serviceaccount/elastic-agent created
```

### Validate the Elastic Agent

```shell
k -n kube-system get pod -l app=elastic-agent
NAME                  READY   STATUS    RESTARTS   AGE
elastic-agent-h8jd4   1/1     Running   0          2m47s
```

## Resources

- https://artifacthub.io/packages/helm/elastic/elasticsearch
- https://artifacthub.io/packages/helm/elastic/kibana