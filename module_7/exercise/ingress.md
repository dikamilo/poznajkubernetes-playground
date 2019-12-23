# Ćwiczenie

Zainstaluj na swoim lokalnym środowisku NGINX Ingress Controller, korzystając z prostej konfiguracji.

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
```

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud-generic.yaml
```

```
k get all -n ingress-nginx
```

```
NAME                                            READY   STATUS    RESTARTS   AGE
pod/nginx-ingress-controller-7dcc95dfbf-dbgcj   1/1     Running   0          47s

NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx   LoadBalancer   10.110.155.130   localhost     80:31505/TCP,443:30041/TCP   43s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-ingress-controller   1/1     1            1           47s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-ingress-controller-7dcc95dfbf   1         1         1       47s
```