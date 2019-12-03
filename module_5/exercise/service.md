# Ćwiczenie 1

Korzystając z wiedzy z lekcji przetestuj następujące scenariusze komunikacji:

- container-to-container w Pod. Wykorzystaj do tego nginx.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-container-to-container
spec:
  initContainers:
    - name: setup-default-page
      image: busybox
      env:
        - name: POD_NAME
          valueFrom:
              fieldRef:
                fieldPath: metadata.name
      command: ['sh', '-c', 'echo "Hello from $(POD_NAME)" > /work-dir/index.html']
      volumeMounts:
        - name: workdir
          mountPath: /work-dir
  containers:
  - name: exercise-container-to-container
    image: nginx
    volumeMounts:
      - name: workdir
        mountPath: /usr/share/nginx/html
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80
        name: http
        protocol: TCP
  - name: tools
    image: giantswarm/tiny-tools
    command: ["/bin/sh"]
    args: ["-c", "sleep 3600"]
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
  volumes:
    - name: workdir
      emptyDir: {}

```

```
k apply -f container-to-container.yml
```

```
k exec -it exercise-container-to-container -c tools sh
```

```
curl localhost
```

```
Hello from exercise-container-to-container
```

- Komunikacja pomiędzy Podami – Pod-to-Pod. Wykorzystaj do tego nginx.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-pod-to-pod-nginx
spec:
  initContainers:
    - name: setup-default-page
      image: busybox
      env:
        - name: POD_NAME
          valueFrom:
              fieldRef:
                fieldPath: metadata.name
      command: ['sh', '-c', 'echo "Hello from $(POD_NAME)" > /work-dir/index.html']
      volumeMounts:
        - name: workdir
          mountPath: /work-dir
  containers:
  - name: exercise-pod-to-pod-nginx
    image: nginx
    volumeMounts:
      - name: workdir
        mountPath: /usr/share/nginx/html
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80
        name: http
        protocol: TCP
  volumes:
    - name: workdir
      emptyDir: {}
---
apiVersion: v1
kind: Pod
metadata:
  name: exercise-pod-to-pod-tools
spec:
  containers:
  - name: exercise-pod-to-pod-tools
    image: giantswarm/tiny-tools
    command: ["/bin/sh"]
    args: ["-c", "sleep 3600"]
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

```
k apply -f pod-to-pod.yml
```

```
k get pod -o wide
```

```
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES
exercise-pod-to-pod-nginx   1/1     Running   0          17s   10.1.0.135   docker-desktop   <none>           <none>
exercise-pod-to-pod-tools   1/1     Running   0          17s   10.1.0.136   docker-desktop   <none>           <none>
```

```
k exec -it exercise-pod-to-pod-tools sh
```

```
curl 10.1.0.135
```

```
Hello from exercise-pod-to-pod-nginx
```

- Wykorzystaj nginx i wystaw go za pomocą serwisu ClusterIP w środku klastra.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-service-cluster-ip-nginx-instance1
  labels:
    app: nginx
spec:
  initContainers:
    - name: setup-default-page
      image: busybox
      env:
        - name: POD_NAME
          valueFrom:
              fieldRef:
                fieldPath: metadata.name
      command: ['sh', '-c', 'echo "Hello from $(POD_NAME)" > /work-dir/index.html']
      volumeMounts:
        - name: workdir
          mountPath: /work-dir
  containers:
  - name: exercise-service-cluster-ip-nginx-instance1
    image: nginx
    volumeMounts:
      - name: workdir
        mountPath: /usr/share/nginx/html
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80
        name: http
        protocol: TCP
  volumes:
    - name: workdir
      emptyDir: {}
---
apiVersion: v1
kind: Pod
metadata:
  name: exercise-service-cluster-ip-nginx-instance2
  labels:
    app: nginx
spec:
  initContainers:
    - name: setup-default-page
      image: busybox
      env:
        - name: POD_NAME
          valueFrom:
              fieldRef:
                fieldPath: metadata.name
      command: ['sh', '-c', 'echo "Hello from $(POD_NAME)" > /work-dir/index.html']
      volumeMounts:
        - name: workdir
          mountPath: /work-dir
  containers:
  - name: exercise-service-cluster-ip-nginx-instance2
    image: nginx
    volumeMounts:
      - name: workdir
        mountPath: /usr/share/nginx/html
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80
        name: http
        protocol: TCP
  volumes:
    - name: workdir
      emptyDir: {}
---
apiVersion: v1
kind: Pod
metadata:
  name: exercise-service-cluster-ip-tools
spec:
  containers:
  - name: exercise-service-cluster-ip-tools
    image: giantswarm/tiny-tools
    command: ["/bin/sh"]
    args: ["-c", "sleep 3600"]
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: exercise-service-cluster-ip-nginx
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    name: http
```

```
k apply -f service-clusterip.yml
```

```
k get pod
```

```
NAME                                          READY   STATUS    RESTARTS   AGE
exercise-service-cluster-ip-nginx-instance1   1/1     Running   0          19s
exercise-service-cluster-ip-nginx-instance2   1/1     Running   0          19s
exercise-service-cluster-ip-tools             1/1     Running   0          19s
```

```
k get service
```

```
NAME                                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
exercise-service-cluster-ip-nginx   ClusterIP   10.104.5.69   <none>        80/TCP    12s
kubernetes                          ClusterIP   10.96.0.1     <none>        443/TCP   12d
```

```
k exec -it exercise-service-cluster-ip-tools sh
```

```bash
/ # curl 10.104.5.69
Hello from exercise-service-cluster-ip-nginx-instance1
/ # curl 10.104.5.69
Hello from exercise-service-cluster-ip-nginx-instance2
/ # curl 10.104.5.69
Hello from exercise-service-cluster-ip-nginx-instance2
/ # curl 10.104.5.69
Hello from exercise-service-cluster-ip-nginx-instance1
/ # curl 10.104.5.69
Hello from exercise-service-cluster-ip-nginx-instance1
/ #
```

- Wykorzystaj nginx i wystaw go na świat za pomocą serwisu NodePort w dwóch opcjach: bez wskazywania portu dla NodePort i ze wskazaniem.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-service-node-port-nginx
  labels:
    app: nginx
spec:
  initContainers:
    - name: setup-default-page
      image: busybox
      env:
        - name: POD_NAME
          valueFrom:
              fieldRef:
                fieldPath: metadata.name
      command: ['sh', '-c', 'echo "Hello from $(POD_NAME)" > /work-dir/index.html']
      volumeMounts:
        - name: workdir
          mountPath: /work-dir
  containers:
  - name: exercise-service-node-port-nginx
    image: nginx
    volumeMounts:
      - name: workdir
        mountPath: /usr/share/nginx/html
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80
        name: http
        protocol: TCP
  volumes:
    - name: workdir
      emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: exercise-service-node-port-with-port
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
    nodePort: 32000
---
apiVersion: v1
kind: Service
metadata:
  name: exercise-service-node-port-without-port
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
```

```
k apply -f service-nodeport.yml
```

```
k get pod
```

```
NAME                               READY   STATUS    RESTARTS   AGE
exercise-service-node-port-nginx   1/1     Running   0          8s
```

```
k get service
```

```
NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
exercise-service-node-port-with-port      NodePort    10.102.102.252   <none>        80:32000/TCP   55s
exercise-service-node-port-without-port   NodePort    10.104.87.44     <none>        80:32605/TCP   55s
kubernetes                                ClusterIP   10.96.0.1        <none>        443/TCP        12d
```

```
curl localhost:32000
```

```
Hello from exercise-service-node-port-nginx
```

```
curl localhost:32605
```

```
Hello from exercise-service-node-port-nginx
```

- Wykorzystaj nginx i wystaw go na świat za pomocą serwisu typu LoadBalancer

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-service-loadbalancer-nginx
  labels:
    app: nginx
spec:
  initContainers:
    - name: setup-default-page
      image: busybox
      env:
        - name: POD_NAME
          valueFrom:
              fieldRef:
                fieldPath: metadata.name
      command: ['sh', '-c', 'echo "Hello from $(POD_NAME)" > /work-dir/index.html']
      volumeMounts:
        - name: workdir
          mountPath: /work-dir
  containers:
  - name: exercise-service-loadbalancer-nginx
    image: nginx
    volumeMounts:
      - name: workdir
        mountPath: /usr/share/nginx/html
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80
        name: http
        protocol: TCP
  volumes:
    - name: workdir
      emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: exercise-service-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
```

```
k apply -f service-loadbalancer.yml
```

```
NAME                                  READY   STATUS     RESTARTS   AGE
exercise-service-loadbalancer-nginx   0/1     Init:0/1   0          4s
```

```
k get service
```

```
NAME                            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
exercise-service-loadbalancer   LoadBalancer   10.104.252.98   localhost     80:30418/TCP   50s
kubernetes                      ClusterIP      10.96.0.1       <none>        443/TCP        12d
```

```
curl localhost
```

```
Hello from exercise-service-loadbalancer-nginx
```

# Ćwiczenie 2

Utwórz dwa Pody z aplikacją helloapp, które mają po jednym wspólnym Label, oraz posiadają oprócz tego inne Label (poniżej przykład).

```
# Pod 1
labels:
  app: helloapp
  ver: v1

# Pod 2
labels:
  app: helloapp
  ver: v1
```

Do tak utworzonych Podów podepnij serwis i sprawdź jak się zachowuje.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-multiple-labels-instance1
  labels:
    app: helloapp
    instance: one
spec:
  containers:
  - name: exercise-multiple-labels-instance1
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
kind: Pod
metadata:
  name: exercise-multiple-labels-instance2
  labels:
    app: helloapp
    instance: two
spec:
  containers:
  - name: exercise-multiple-labels-instance2
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
  name: exercise-multiple-labels
spec:
  type: NodePort
  selector:
    app: helloapp
  ports:
  - port: 8080
    name: http
```

```
k apply -f multiple-labels.yml
```

```
k get pod --show-labels=true
```

```
NAME                                 READY   STATUS    RESTARTS   AGE     LABELS
exercise-multiple-labels-instance1   1/1     Running   0          2m50s   app=helloapp,instance=one
exercise-multiple-labels-instance2   1/1     Running   0          2m50s   app=helloapp,instance=two
```

```
k get service
```

```
NAME                       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
exercise-multiple-labels   NodePort    10.104.70.86   <none>        8080:31367/TCP   3m3s
kubernetes                 ClusterIP   10.96.0.1      <none>        443/TCP          13d
```

```
curl localhost:31367
```

```
Cześć, 🚢 =>  exercise-multiple-labels-instance1
Cześć, 🚢 =>  exercise-multiple-labels-instance2
Cześć, 🚢 =>  exercise-multiple-labels-instance1
```

Service poprawnie rozdziela ruch między podami z etykietą `app=helloapp` pomimo tego że pody mają przypisaną również inną etykietę. W przypadku zmiany selektora na:

```yaml
selector:
  app: helloapp
  instance: one
```

Ruch będzie przekazywany tylko do pierwszej instancji:

```
Cześć, 🚢 =>  exercise-multiple-labels-instance1
Cześć, 🚢 =>  exercise-multiple-labels-instance1
Cześć, 🚢 =>  exercise-multiple-labels-instance1
```
