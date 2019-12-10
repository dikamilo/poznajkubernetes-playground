# Ćwiczenie

Korzystając z wiedzy na temat rolling update przetestuj:

1. Działanie trybu Recreate

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recreate-deployment
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: recreate-deployment
  template:
    metadata:
      labels:
        app: recreate-deployment
    spec:
      containers:
      - name: recreate-deployment
        image: poznajkubernetes/pkad:blue
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
        env:
        - name: version
          value: v1
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
```

```
k apply -f recreate.yml
```

```
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
recreate-deployment   3/3     3            3           18s
```

```
NAME                            DESIRED   CURRENT   READY   AGE
recreate-deployment-6d6d6c54d   3         3         3       26s
```

```
NAME                                  READY   STATUS    RESTARTS   AGE
recreate-deployment-6d6d6c54d-28qtr   1/1     Running   0          35s
recreate-deployment-6d6d6c54d-62n7n   1/1     Running   0          35s
recreate-deployment-6d6d6c54d-65jbm   1/1     Running   0          35s
```

```
k apply -f recreate.yml
```

```
NAME                                  READY   STATUS        RESTARTS   AGE
recreate-deployment-6d6d6c54d-28qtr   0/1     Terminating   0          66s
recreate-deployment-6d6d6c54d-62n7n   0/1     Terminating   0          66s
recreate-deployment-6d6d6c54d-65jbm   0/1     Terminating   0          66s
```

```
NAME                                   READY   STATUS    RESTARTS   AGE
recreate-deployment-74bbf7568f-flq64   0/1     Running   0          7s
recreate-deployment-74bbf7568f-vz9l5   0/1     Running   0          7s
recreate-deployment-74bbf7568f-wf4pq   1/1     Running   0          7s
```

```
NAME                             DESIRED   CURRENT   READY   AGE
recreate-deployment-6d6d6c54d    0         0         0       106s
recreate-deployment-74bbf7568f   3         3         3       38s
```

Wszystkie pody są ubijane a następnie tworzone są nowe.


2. Szybki i wolny rolling update

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: slow-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: slow-deployment
  template:
    metadata:
      labels:
        app: slow-deployment
    spec:
      containers:
      - name: slow-deployment
        image: poznajkubernetes/pkad:blue
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
        env:
        - name: version
          value: v1
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
```

```
k apply -f slow.yml
```

```
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
slow-deployment   3/3     3            3           21s
```

```
NAME                         DESIRED   CURRENT   READY   AGE
slow-deployment-85b8d46d56   3         3         3       29s
```

```
k apply -f slow.yml
```

```
NAME                         DESIRED   CURRENT   READY   AGE
slow-deployment-6cfd69b677   3         3         3       24s
slow-deployment-85b8d46d56   0         0         0       73s
```

K8s podmienia pody jeden po drugim, wyłączając tylko jeden Pod ze starego ReplicaSet.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fast-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 1
  selector:
    matchLabels:
      app: fast-deployment
  template:
    metadata:
      labels:
        app: fast-deployment
    spec:
      containers:
      - name: fast-deployment
        image: poznajkubernetes/pkad:blue
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
        env:
        - name: version
          value: v1
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
```

```
k apply -f fast.yml
```

```
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
fast-deployment   3/3     3            3           17s
```

```
NAME                         DESIRED   CURRENT   READY   AGE
fast-deployment-6bbf6fb656   3         3         3       22s
```

```
k apply -f fast.yml
```

```
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
fast-deployment   2/3     3            2           51s
```

```
NAME                         DESIRED   CURRENT   READY   AGE
fast-deployment-6bbf6fb656   0         0         0       58s
fast-deployment-6fc4c54c7    3         3         3       18s
```

K8s uruchamia 3 nowe Pody, a następnie któryś z nich jest gotowy, zostaje podmienony jeden po drugim.


3. Zmiany na deployment bez liveness i readiness

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: noprobes-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 1
  selector:
    matchLabels:
      app: noprobes-deployment
  template:
    metadata:
      labels:
        app: noprobes-deployment
    spec:
      containers:
      - name: noprobes-deployment
        image: poznajkubernetes/pkad:blue
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
        env:
        - name: version
          value: v1
```

```
k apply -f no-probes.yml
```

```
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
noprobes-deployment   3/3     3            3           15s
```

```
NAME                             DESIRED   CURRENT   READY   AGE
noprobes-deployment-777667b779   3         3         3       23s
```

```
k apply -f no-probes.yml
```

```
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
noprobes-deployment   3/3     3            3           62s
```

```
NAME                             DESIRED   CURRENT   READY   AGE
noprobes-deployment-68c4587fbd   3         3         3       13s
noprobes-deployment-777667b779   0         0         0       69s
```

Pody są dostępne praktycznie od razu po utworzeniu co może powodować HTTP 503 gdy aplikacja jeszcze nie jest gotowa na przyjmowanie ruchu.


4. Zmiany na deployment z działającym readiness

Sprawdzone w poprzednich ćwiczeniach ;) Pod zostanie podmieniony gdy jest gotowy na przyjmowanie ruchu.


5. Zmiany na deployment z niedziałającym readiness (podaj np. błędny endpoint do sprawdzania)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken-readiness-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 1
  selector:
    matchLabels:
      app: broken-readiness-deployment
  template:
    metadata:
      labels:
        app: broken-readiness-deployment
    spec:
      containers:
      - name: broken-readiness-deployment
        image: poznajkubernetes/pkad:blue
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
        env:
        - name: version
          value: v1
        readinessProbe:
          httpGet:
            path: /wrongurl
            port: 8080
```

Uruchomienie z poprawnym readiness aby pierwszy deployment się wykonał poprawnie:

```
k apply -f broken-readiness.yml
```

```
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
broken-readiness-deployment   3/3     3            3           10s
```

```
NAME                                     DESIRED   CURRENT   READY   AGE
broken-readiness-deployment-75676d5699   3         3         3       17s
```

```
k apply -f broken-readiness.yml
```

```
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
broken-readiness-deployment   2/3     3            2           55s
```

```
NAME                                     DESIRED   CURRENT   READY   AGE
broken-readiness-deployment-75676d5699   2         2         2       60s
broken-readiness-deployment-cb54565fc    3         3         0       27s
```

```
k rollout status deployment broken-readiness-deployment
```

```
Waiting for deployment "broken-readiness-deployment" rollout to finish: 2 old replicas are pending termination...
```

Deployment nie postępuje, cały czas oczekuje readiness od nowych podów.
