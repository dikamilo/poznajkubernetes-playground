# Ćwiczenie

Zainstaluj Ingress Controller na swoim klastrze i:

1. Stwórz Ingress (i pomocnicze zasoby) jako default backend

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exercise-ingress-default-backend
spec:
  selector:
    matchLabels:
      app: exercise-ingress-default-backend
  template:
    metadata:
      labels:
        app: exercise-ingress-default-backend
    spec:
      containers:
      - name: exercise-ingress-default-backend
        image: poznajkubernetes/pkad:blue
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
          name: http
---
apiVersion: v1
kind: Service
metadata:
  name: exercise-ingress-default-backend-service
spec:
  selector:
    app: exercise-ingress-default-backend
  ports:
  - port: 80
    targetPort: http
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: exercise-ingress-default-backend-ingress
spec:
  backend:
    serviceName: exercise-ingress-default-backend-service
    servicePort: 80
```

```
k apply -f default-backend.yml
```

```
k get pod
```

```
NAME                                               READY   STATUS    RESTARTS   AGE
exercise-ingress-default-backend-569cd7c98-p44j9   1/1     Running   0          4s
```

```
k get svc
```

```
NAME                                       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
exercise-ingress-default-backend-service   ClusterIP   10.99.138.98   <none>        80/TCP    7s
kubernetes                                 ClusterIP   10.96.0.1      <none>        443/TCP   32d
```

```
k get ingress
```

```
NAME                                       HOSTS   ADDRESS     PORTS   AGE
exercise-ingress-default-backend-ingress   *       localhost   80      4m58s
```

2. Stwórz Ingress (i pomocnicze zasoby) i ustaw fanout routing

```yml
apiVersion: v1
kind: Service
metadata:
  name: exercise-fanout-routing-echo-service
spec:
  selector:
    app: exercise-fanout-routing-echo
  ports:
  - port: 8080
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exercise-fanout-routing-echo
spec:
  selector:
    matchLabels:
      app: exercise-fanout-routing-echo
  template:
    metadata:
      labels:
        app: exercise-fanout-routing-echo
    spec:
      containers:
      - name: exercise-fanout-routing-echo
        image: gcr.io/google_containers/echoserver:1.4
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
```

```yml
apiVersion: v1
kind: Service
metadata:
  name: exercise-fanout-pkad-service
spec:
  selector:
    app: exercise-fanout-pkad
  ports:
  - port: 8080
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exercise-fanout-pkad
spec:
  selector:
    matchLabels:
      app: exercise-fanout-pkad
  template:
    metadata:
      labels:
        app: exercise-fanout-pkad
    spec:
      containers:
      - name: exercise-fanout-pkad
        image: poznajkubernetes/helloapp:svc
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
```

```yml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: exercise-fanout
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
  - http:
      paths:
      - path: /pkad/?(.*)
        backend:
          serviceName: exercise-fanout-pkad-service
          servicePort: 8080
      - path: /echo/?(.*)
        backend:
          serviceName: exercise-fanout-routing-echo-service
          servicePort: 8080
```

```
k apply -f fanout-echo.yml
k apply -f fanout-pkad.yml
k apply -f fanout.yml
```

```
curl localhost/pkad
```

```
Cześć, 🚢 =>  exercise-fanout-pkad-5fbf45f7cf-7vbrg
```

```
curl localhost/echo/poznajkubernetes
```

```
CLIENT VALUES:
client_address=10.1.0.225
command=GET
real path=/poznajkubernetes
query=nil
request_version=1.1
request_uri=http://localhost:8080/poznajkubernetes

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=localhost
user-agent=curl/7.58.0
x-forwarded-for=192.168.65.3
x-forwarded-host=localhost
x-forwarded-port=80
x-forwarded-proto=http
x-real-ip=192.168.65.3
x-request-id=590752e12ecb67afa44cef0f8a502af4
x-scheme=http
BODY:
-no body in request-
```

3. Stwórz Ingress (i pomocnicze zasoby) i ustaw host routing

```yml
apiVersion: v1
kind: Service
metadata:
  name: exercise-fanout-routing-echo-service
spec:
  selector:
    app: exercise-fanout-routing-echo
  ports:
  - port: 8080
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exercise-fanout-routing-echo
spec:
  selector:
    matchLabels:
      app: exercise-fanout-routing-echo
  template:
    metadata:
      labels:
        app: exercise-fanout-routing-echo
    spec:
      containers:
      - name: exercise-fanout-routing-echo
        image: gcr.io/google_containers/echoserver:1.4
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
```

```yml
apiVersion: v1
kind: Service
metadata:
  name: exercise-fanout-pkad-service
spec:
  selector:
    app: exercise-fanout-pkad
  ports:
  - port: 8080
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exercise-fanout-pkad
spec:
  selector:
    matchLabels:
      app: exercise-fanout-pkad
  template:
    metadata:
      labels:
        app: exercise-fanout-pkad
    spec:
      containers:
      - name: exercise-fanout-pkad
        image: poznajkubernetes/helloapp:svc
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
```

```yml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: exercise-fanout-host
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: pkad.127.0.0.1.nip.io
      http:
        paths:
          - backend:
              serviceName: exercise-fanout-pkad-service
              servicePort: 8080
    - host: echo.127.0.0.1.nip.io
      http:
        paths:
          - backend:
              serviceName: exercise-fanout-routing-echo-service
              servicePort: 8080
```

```
k apply -f fanout-echo.yml
k apply -f fanout-pkad.yml
k apply -f fanout-host.yml
```

```
curl pkad.127.0.0.1.nip.io
```

```
Cześć, 🚢 =>  exercise-fanout-pkad-5fbf45f7cf-7vbrg
```

```
curl echo.127.0.0.1.nip.io/poznajkubernetes/
```

```
CLIENT VALUES:
client_address=10.1.0.225
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://echo.127.0.0.1.nip.io:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=echo.127.0.0.1.nip.io
user-agent=curl/7.58.0
x-forwarded-for=192.168.65.3
x-forwarded-host=echo.127.0.0.1.nip.io
x-forwarded-port=80
x-forwarded-proto=http
x-real-ip=192.168.65.3
x-request-id=7a832200529775c451ad871d82f91d84
x-scheme=http
BODY:
-no body in request-
```

4. Stwórz Ingress (i pomocnicze zasoby) i wymieszaj dowolnie fanout i host routing

```yml
apiVersion: v1
kind: Service
metadata:
  name: exercise-fanout-routing-echo-service
spec:
  selector:
    app: exercise-fanout-routing-echo
  ports:
  - port: 8080
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exercise-fanout-routing-echo
spec:
  selector:
    matchLabels:
      app: exercise-fanout-routing-echo
  template:
    metadata:
      labels:
        app: exercise-fanout-routing-echo
    spec:
      containers:
      - name: exercise-fanout-routing-echo
        image: gcr.io/google_containers/echoserver:1.4
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
```

```yml
apiVersion: v1
kind: Service
metadata:
  name: exercise-fanout-pkad-service
spec:
  selector:
    app: exercise-fanout-pkad
  ports:
  - port: 8080
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exercise-fanout-pkad
spec:
  selector:
    matchLabels:
      app: exercise-fanout-pkad
  template:
    metadata:
      labels:
        app: exercise-fanout-pkad
    spec:
      containers:
      - name: exercise-fanout-pkad
        image: poznajkubernetes/helloapp:svc
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
```

```yml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: exercise-fanout-host
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: 127.0.0.1.nip.io
      http:
        paths:
          - path: /pkad/?(.*)
            backend:
              serviceName: exercise-fanout-pkad-service
              servicePort: 8080
          - backend:
              serviceName:  exercise-fanout-routing-echo-service
              servicePort: 8080
```

```
curl 127.0.0.1.nip.io/pkad/test/
```

```
Cześć, 🚢 =>  exercise-fanout-pkad-5fbf45f7cf-7vbrg
```

```
curl 127.0.0.1.nip.io/test
```

```
CLIENT VALUES:
client_address=10.1.0.225
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://127.0.0.1.nip.io:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=127.0.0.1.nip.io
user-agent=curl/7.58.0
x-forwarded-for=192.168.65.3
x-forwarded-host=127.0.0.1.nip.io
x-forwarded-port=80
x-forwarded-proto=http
x-real-ip=192.168.65.3
x-request-id=f8f3db1c078891148f0084ece891de5e
x-scheme=http
BODY:
-no body in request-
```
