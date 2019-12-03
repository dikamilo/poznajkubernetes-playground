# Ćwiczenie

Przetestuj działanie Service Discovery korzystając ze swojej aplikacji albo helloapp.

- Utwórz dwa namespaces

```
k create ns first
```

```
k create ns second
```

```
k get namespaces
```

```
NAME              STATUS   AGE
default           Active   13d
docker            Active   13d
first             Active   23s
kube-node-lease   Active   13d
kube-public       Active   13d
kube-system       Active   13d
second            Active   18s
```


- W każdym namespaces umieć pod i serwis

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-namespace
  labels:
    app: helloapp
spec:
  containers:
  - name: exercise-namespace
    image: poznajkubernetes/helloapp:svc
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: exercise-namespace
spec:
  type: ClusterIP
  selector:
    app: helloapp
  ports:
  - port: 8080
    name: http
```

```
k apply -f namespace.yml -n first
```

```
k apply -f namespace.yml -n second
```

```
k get pod -A
```

```
NAMESPACE     NAME                                     READY   STATUS    RESTARTS   AGE
first         exercise-namespace                       1/1     Running   0          6m9s
kube-system   coredns-6dcc67dcbc-jc89n                 1/1     Running   7          13d
kube-system   coredns-6dcc67dcbc-zl94d                 1/1     Running   7          13d
kube-system   etcd-docker-desktop                      1/1     Running   8          13d
kube-system   kube-apiserver-docker-desktop            1/1     Running   8          13d
kube-system   kube-controller-manager-docker-desktop   1/1     Running   8          13d
kube-system   kube-proxy-fvwq5                         1/1     Running   7          13d
kube-system   kube-scheduler-docker-desktop            1/1     Running   8          13d
second        exercise-namespace                       1/1     Running   0          5m46s
```

```
k get service -A
```

```
NAMESPACE     NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes                 ClusterIP   10.96.0.1       <none>        443/TCP                  13d
docker        compose-api                ClusterIP   10.111.135.82   <none>        443/TCP                  13d
first         exercise-namespace         ClusterIP   10.97.105.170   <none>        8080/TCP                 7m13s
kube-system   kube-dns                   ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   13d
second        exercise-namespace         ClusterIP   10.111.18.205   <none>        8080/TCP                 6m50s
```

```
k get service -n first
```

```
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
exercise-namespace   ClusterIP   10.97.105.170   <none>        8080/TCP   60s
```

```
k get service -n second
```

```
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
exercise-namespace   ClusterIP   10.111.18.205   <none>        8080/TCP   41s
```

- Przetestuj działanie Service Discovery z wykorzystaniem curl i nslookup. Jeśli używasz swojej aplikacji wywołaj endpointy pomiędzy aplikacjami.

```
kubectl run -it --rm tools --generator=run-pod/v1 --image=giantswarm/tiny-tools
```

```bash
/ # nslookup exercise-namespace
Server:         10.96.0.10
Address:        10.96.0.10#53
```

```
/ # nslookup exercise-namespace.first
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   exercise-namespace.first.svc.cluster.local
Address: 10.97.105.170
```

```
/ # nslookup exercise-namespace.second
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   exercise-namespace.second.svc.cluster.local
Address: 10.111.18.205
```

```
/ # curl exercise-namespace:8080
curl: (6) Could not resolve host: exercise-namespace
```

```
/ # curl exercise-namespace.first:8080
Cześć, 🚢 =>  exercise-namespace
```

```
/ # curl exercise-namespace.second:8080
Cześć, 🚢 =>  exercise-namespace
```