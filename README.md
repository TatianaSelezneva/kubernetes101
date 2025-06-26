# Kubernetes 101

free playground: https://kodekloud.com/public-playgrounds/multi-node-k8s-1-33/

Options: https://learn.kodekloud.com/user/courses/kubernetes-challenges
Challenges are ree, but sign up is required

Lab 1 - RBAC
Lab 2 - broken cluster, troobleshooting
Lab3 - deployments and services
Lab4 - highly available Redis Cluster



```
kubectl --help
```

alias for kubectl  = `k`

```
k get nodes
```

## Namespaces

```
k create ns k101
```

List all the namespaces now, you should see our namespace in the list

```
k get ns
```

## Pods

```
vim pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: k101
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

```
k apply -f pod.yaml
```

```
k get pods
```
Expect no pods found

Try with namespace:

```
k get pods -n k101
```

## Depolyments

```
vim deploy.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: k101
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
        ports:
        - containerPort: 80
```

Create deployment:
```
k apply -f deploy.yaml
```

```
k get pods -n k101 --show-labels
```

```
k -n k101 delete pod nginx
```

```
k -n k101 get pods
```


```
k -n k101 delete pod nginx-deployment(press tab to use autocomplete)
```

Watch pod being re-created with a different name

## Service

```
vim service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: k101
spec:
  selector:
    app: nginx
  ports:
    - port: 80          # Port exposed by the service
      targetPort: 80    # Port on the Pod
      protocol: TCP     # TCP (default), UDP, or SCTP
  type: ClusterIP       # Service type
  ```

```
k run test --image=nginx -it --rm -n k101 -- sh
```

```
curl my-service
```

### Kubernetes DNS Naming Convention

The URL my-service.k101.svc.cluster.local follows Kubernetes' DNS naming convention and breaks down as follows:

`my-service` - The name of the Service resource as defined in your service.yaml
`k101` - The namespace where the Service is deployed
`svc` - Indicates this is a Service resource type
`cluster.local` - The default DNS domain for the Kubernetes cluster

This follows the standard Kubernetes DNS pattern:

```
<service-name>.<namespace>.svc.cluster.local
```

This DNS structure allows pods within the cluster to reliably communicate with services using consistent naming, regardless of the actual pod IPs or locations. The example shows accessing the nginx service from a test pod created in the same namespace.

You can also use shorter forms within the same namespace (just my-service) or from other namespaces (my-service.k101)


## Scaling

```
vim hpa.yaml
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: k101-hpa
  namespace: k101
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

```
k apply -f hpa.yaml
```

We need metrics server worker, which we don't have in the playground.

Install metrics server to see it working:

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

```
kubectl patch -n kube-system deployment metrics-server --type=json -p '[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```

-------

Wifi:
networks: Loungeroom
username: rgroup4
password: 8605

-------

Vim help:

To enter insert mode: i
To save and exit: :x
To exit no saving :q!
