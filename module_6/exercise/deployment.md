# Ćwiczenie

Przetestuj każde z pytań w praktyce!

Pytania:

1. Co się stanie kiedy dodasz Pod spełniający selektor ReplicaSet?

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-additional-pod
spec:
  replicas: 2
  selector:
    matchLabels:
      app: replicaset-test
  template:
    metadata:
      labels:
        app: replicaset-test
    spec:
      containers:
      - name: replicaset-additional-pod
        image: poznajkubernetes/helloapp:svc
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: additional-pod
  labels:
    app: replicaset-test
spec:
  containers:
  - name: additional-pod
    image: poznajkubernetes/helloapp:svc
    ports:
    - containerPort: 8080
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

```
k apply -f replicaset-additional-pod.yml
```

```
NAME                        DESIRED   CURRENT   READY   AGE
replicaset-additional-pod   2         2         2       17s
```

```
NAME                              READY   STATUS    RESTARTS   AGE
replicaset-additional-pod-rs4m6   1/1     Running   0          24s
replicaset-additional-pod-whx46   1/1     Running   0          24s
```

```
k apply -f additional-pod.yml
```

```
NAME                              READY   STATUS        RESTARTS   AGE
additional-pod                    0/1     Terminating   0          3s
replicaset-additional-pod-rs4m6   1/1     Running       0          102s
replicaset-additional-pod-whx46   1/1     Running       0          102s
```

Dodatkowy pod dostaje od razu status `Terminating` ponieważ ReplicaSet pilnuje aby były tylko 2 repliki tego poda, które już są uruchomione.

2. Jak zadziała minReadySeconds bez readiness i liveliness probes?

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-probes
spec:
  replicas: 2
  minReadySeconds: 10
  selector:
    matchLabels:
      app: deployment-probes
  template:
    metadata:
      labels:
        app: deployment-probes
    spec:
      containers:
      - name: deployment-probes
        image: poznajkubernetes/helloapp:svc
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
```

```
k apply -f deployment-selector.yml
```

```
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
deployment-probes   2/2     2            2           17s
```

Deployment uznaje wszystkie pody za dostępne w momencie przekroczenia `minReadySeconds`.


3. Do czego może Ci się przydać matchExpressions?

Może się przydać do filtrowania jakie pody mają nie być zarządzane przez Deployment oraz przez ReplicaSet. Na przykład Deployment ma skalować aplikację bez etykiet które mogą być wykorzystane w podach które są uruchamiane ręcznie w celu diagnostycznych np. aplikacja w trybie debug, jednorazowa operacja w podzie aplikacji, cron job itp.


4. Jak najlepiej (według Ciebie) zarządzać historią zmian w deploymentach?

CI/DI które przy deploymencie będzie wysyłać powiadomienie np. na kanał slack z informacją co jest deployowane wraz ze zmianami np. nazwa obrazu.


5. Co się stanie jak usuniesz ReplicaSet stworzony przez Deployment?

```
k apply -f deployment-probes.yml
```

```
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
deployment-probes   2/2     2            2           35s
```

```
NAME                           DESIRED   CURRENT   READY   AGE
deployment-probes-6dc8b7d767   2         2         2       40s
```

```
NAME                                 READY   STATUS    RESTARTS   AGE
deployment-probes-6dc8b7d767-9jkwt   1/1     Running   0          14s
deployment-probes-6dc8b7d767-mfz8n   1/1     Running   0          14s
```

```
k delete rs deployment-probes-6dc8b7d767
```

```
NAME                           DESIRED   CURRENT   READY   AGE
deployment-probes-6dc8b7d767   2         2         2       25s
```

```
NAME                                 READY   STATUS        RESTARTS   AGE
deployment-probes-6dc8b7d767-9jkwt   0/1     Terminating   0          30s
deployment-probes-6dc8b7d767-mfz8n   0/1     Terminating   0          30s
deployment-probes-6dc8b7d767-qvdzh   1/1     Running       0          2s
deployment-probes-6dc8b7d767-tpvq9   1/1     Running       0          2s
```

Usunięcie ReplicaSet przed deploymentem usuwa tą ReplicaSet wraz podami, jednak Deployment niemal natychmiast tworzy nową ReplicaSet i uruchamia pody ponownie. 


6. Czy Pod może definiować więcej etykiet niż ReplicaSet ma zdefiniowane w selectorze?

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-multiple-labels
spec:
  replicas: 2
  selector:
    matchLabels:
      app: replicaset-multiple-labels
  template:
    metadata:
      labels:
        app: replicaset-multiple-labels
        env: production
    spec:
      containers:
      - name: replicaset-multiple-labels
        image: poznajkubernetes/helloapp:svc
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
```

```
k apply -f replicaset-multiple-labels.yml
```

```
NAME                         DESIRED   CURRENT   READY   AGE
replicaset-multiple-labels   2         2         2       32s
```

```
NAME                               READY   STATUS    RESTARTS   AGE
replicaset-multiple-labels-99pcx   1/1     Running   0          38s
replicaset-multiple-labels-fr2qm   1/1     Running   0          38s
```

Tak, Pod może posiadać więcej etykiet niż te zdefiniowane w ReplicaSet.

7. Czy ReplicaSet może definiować więcej etykiet w selektorze niz Pod ma zdefiniowane?

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-multiple-labels
spec:
  replicas: 2
  selector:
    matchLabels:
      app: replicaset-multiple-labels
      env: production
  template:
    metadata:
      labels:
        app: replicaset-multiple-labels
    spec:
      containers:
      - name: replicaset-multiple-labels
        image: poznajkubernetes/helloapp:svc
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
```

```
k apply -f replicaset-multiple-labels.yml
```

```
The ReplicaSet "replicaset-multiple-labels" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"app":"replicaset-multiple-labels"}: `selector` does not match template `labels`
```

Nie, ponieważ ReplicaSet nie znajdzie wtedy Podów którymi ma zarządzać.