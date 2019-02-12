# Working With Helm and Kubernetes

I have Kubernetes setup.


```
$ helm init
```
Results:

```
$HELM_HOME has been configured at /root/.helm.
Warning: Tiller is already installed in the cluster.
(Use --client-only to suppress this message, or --upgrade to upgrade Tiller to the current version.)
Happy Helming!
```

Now to validate `Tiller.`

```
$ kubectl --namespace kube-system get pods | grep tiller
tiller-deploy-776b5cb874-nwlmp       1/1     Running   0          9d
```

## Create a chart

```
$ helm create mychart
Creating mychart
```


```
$ cd mychart/
$ tree .
.
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
3 directories, 8 files
```

## Building the Starter Helm Project

```
$ helm install --dry-run --debug ./mychart
```

### The values.yml file

Contains the `values` that fills in the templates.

**values.yml**

```
# Default values for mychart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.
---
# for web-pod-1.yml
metadata:
  labels:
    app: demo
    name: web
  name: web1
containers:
  image1: redis:latest
  name1: redis
  containerPort1: 6379
  image2: brunoterkaly/py-red
  name2: python
  env:
    name: REDIS_HOST
    value: localhost
    containerPort: 5000

# for web-svc-1.yml
webservice:
  type: LoadBalancer
  port: 80
  targetPort: 5000

# for db-pod-1.yml
mysqlpod:
  name: mysql
  label_name: mysql
  label_app: demo
  container_name: mysql
  container_image: mysql:5.7
  container_port: 3306
  env_name: "MYSQL_ROOT_PASSWORD"
  env_value: "password"

# for db-svc-1.yml
mysqlsvc:
  name: mysql
  label_name: mysql
  label_app: demo
  port_name: mysql
  port_port: 3306
  port_targetport: 3306
  selector_name: mysql
  selector_app: demo
```

**web-pod-1.yml**

```
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: {{ .Values.metadata.labels.app }}
    name: {{ .Values.metadata.labels.name }}
  name: {{ .Values.metadata.name }}
spec:
  containers:
    - image: {{ .Values.containers.image1 }}
      name: {{ .Values.containers.name1 }}
      ports:
        - containerPort: {{ .Values.containers.containerPort1 }}
          name: {{ .Values.containers.name1 }}
          protocol: TCP
    - image: {{ .Values.containers.image2 }}
      name: {{ .Values.containers.name2 }}
      env:
        - name: {{ .Values.containers.env.name }}
          value: {{ .Values.containers.env.value }}
      ports:
        - containerPort: {{ .Values.containers.env.containerPort }}
          name: http
          protocol: TCP
```

**web-svc-1.yml**

```
apiVersion: v1
kind: Service
metadata:
  name: web
  labels:
    app: {{ .Values.metadata.labels.app }}
    name: {{ .Values.metadata.labels.name }}
spec:
  selector:
    name: {{ .Values.metadata.labels.name }}
  type: {{ .Values.webservice.type }}
  ports:
   - port: {{ .Values.webservice.port }}
     name: http
     targetPort: {{ .Values.webservice.targetPort }}
     protocol: TCP```
```

**db-pod-1.yml**

```
apiVersion: "v1"
kind: Pod
metadata:
  name: {{ .Values.mysqlpod.name }}
  labels:
    name: {{ .Values.mysqlpod.label_name }}
    app: {{ .Values.mysqlpod.label_app }}
spec:
  containers:
    - name: {{ .Values.mysqlpod.container_name }}
      image: {{ .Values.mysqlpod.container_image }}
      ports:
        - containerPort: {{ .Values.mysqlpod.container_port }}
          protocol: TCP
      env:
        -
          name: {{ .Values.mysqlpod.env_name }}
          value: {{ .Values.mysqlpod.env_value }}
```

**db-svc-1.yml**

```
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.mysqlsvc.name }}
  labels:
    name: {{ .Values.mysqlsvc.label_name }}
    app: {{ .Values.mysqlsvc.label_app }}
spec:
  ports:
    - name: {{ .Values.mysqlsvc.port_name }}
      port: {{ .Values.mysqlsvc.port_port }}
      targetPort: {{ .Values.mysqlsvc.port_targetport }}
  selector:
    name: {{ .Values.mysqlsvc.selector_name }}
    app: {{ .Values.mysqlsvc.selector_app }}
```

### Packaging the Helm Chartrt

```
$ helm package mychart --debug
```


### Installing the Helm Chart

```
$  helm install mychart-0.1.0.tgz
------------------------------------------
NAME:   quarreling-koala
LAST DEPLOYED: Tue Feb 12 15:46:08 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME   TYPE          CLUSTER-IP   EXTERNAL-IP  PORT(S)       AGE
mysql  ClusterIP     10.0.242.37  <none>       3306/TCP      1s
web    LoadBalancer  10.0.9.166   <pending>    80:30186/TCP  1s

==> v1/Pod
NAME   READY  STATUS             RESTARTS  AGE
mysql  0/1    ContainerCreating  0         1s
web1   0/2    ContainerCreating  0         1s
```

> You will need the name for deletions: `NAME:   quarreling-koala`


### Deleting the Helm deployment

```
$ helm delete quarreling-koala
-----------------------------------
release "quarreling-koala" deleted
```