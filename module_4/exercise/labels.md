# Ćwiczenie 1

Wykonaj podstawowe operacje na etykietach imperatywnie. Takie operacje będą przydatne w późniejszych częściach szkolenia jak na przykład trzeba będzie przeanalizować niedziałający Pod albo przekierować ruch na inne Pody.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-labels-first-pod
spec:
  containers:
  - name: exercise-labels-first-pod
    image: poznajkubernetes/pkad
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
---
apiVersion: v1
kind: Pod
metadata:
  name: exercise-labels-second-pod
spec:
  containers:
  - name: exercise-labels-second-pod
    image: poznajkubernetes/pkad
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

```
kubectl apply -f pod-labels.yml
```

- Dodaj etykietę

```
kubectl label pods exercise-labels-first-pod env=test
```

```
kubectl get pods --show-labels
```

```
NAME                         READY   STATUS    RESTARTS   AGE   LABELS
exercise-labels-first-pod    1/1     Running   0          75s   env=test
exercise-labels-second-pod   1/1     Running   0          75s   <none>
```

- Dodaj etykietę do wszystkich zasobów na raz

```
kubectl label pods --all a=b
```

```
kubectl get pods --show-labels
```

```
NAME                         READY   STATUS    RESTARTS   AGE     LABELS
exercise-labels-first-pod    1/1     Running   0          2m26s   a=b,env=test
exercise-labels-second-pod   1/1     Running   0          2m26s   a=b
```

- Zaktualizuj etykietę

```
kubectl label pods exercise-labels-first-pod --overwrite env=stage
```

```
kubectl get pods --show-labels
```

```
NAME                         READY   STATUS    RESTARTS   AGE     LABELS
exercise-labels-first-pod    1/1     Running   0          3m15s   a=b,env=stage
exercise-labels-second-pod   1/1     Running   0          3m15s   a=b
```

- Usuń etykietę

```
kubectl label pods --all a-
```

```
kubectl get pods --show-labels
```

```
NAME                         READY   STATUS    RESTARTS   AGE    LABELS
exercise-labels-first-pod    1/1     Running   0          4m4s   env=stage
exercise-labels-second-pod   1/1     Running   0          4m4s   <none>
```

# Ćwiczenie 2

Stwórz trzy Pody z czego dwa posiadające po dwie etykiety: app=ui i env=test oraz app=ui i env=stg, trzeci bez etykiet.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-labels-first-pod
  labels:
    app: ui
    env: test
spec:
  containers:
  - name: exercise-labels-first-pod
    image: poznajkubernetes/pkad
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
---
apiVersion: v1
kind: Pod
metadata:
  name: exercise-labels-second-pod
  labels:
    app: ui
    env: stg
spec:
  containers:
  - name: exercise-labels-second-pod
    image: poznajkubernetes/pkad
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
---
apiVersion: v1
kind: Pod
metadata:
  name: exercise-labels-third-pod
spec:
  containers:
  - name: exercise-labels-third-pod
    image: poznajkubernetes/pkad
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

```
kubectl apply -f pod-labels-selector.yml
```

```
kubectl get pods --show-labels
```

```
NAME                         READY   STATUS    RESTARTS   AGE   LABELS
exercise-labels-first-pod    1/1     Running   0          10s   app=ui,env=test
exercise-labels-second-pod   1/1     Running   0          10s   app=ui,env=stg
exercise-labels-third-pod    1/1     Running   0          10s   <none>
```

- Wybierz wszystkie Pody które mają etykietę env zdefiniowaną

```
kubectl get pods --selector env --show-labels
```

```
NAME                         READY   STATUS    RESTARTS   AGE     LABELS
exercise-labels-first-pod    1/1     Running   0          2m52s   app=ui,env=test
exercise-labels-second-pod   1/1     Running   0          2m52s   app=ui,env=stg
```

- Wybierz wszystkie Pody które nie mają etykiety env zdefiniowanej

```
kubectl get pods --selector '!env' --show-labels
```

```
NAME                        READY   STATUS    RESTARTS   AGE     LABELS
exercise-labels-third-pod   1/1     Running   0          2m41s   <none>
```

- Wybierz Pody które mają app=ui ale nie znajdują się w env=stg

```
kubectl get pods --selector app=ui,env!=stg --show-labels
```

```
NAME                        READY   STATUS    RESTARTS   AGE     LABELS
exercise-labels-first-pod   1/1     Running   0          6m25s   app=ui,env=test
```

- Wybierz Pody których env znajduje się w przedziale stg i demo

```
kubectl get pods --selector 'env in (stg, demo)' --show-labels
```

```
NAME                         READY   STATUS    RESTARTS   AGE     LABELS
exercise-labels-second-pod   1/1     Running   0          7m13s   app=ui,env=stg
```

# Ćwiczenie 3

Z wcześniej stworzonych Podów:

- Wybierz i wyświetl tylko nazwy Poda

```
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
```

```
exercise-labels-first-pod
exercise-labels-second-pod
exercise-labels-third-pod
```

- Posortuj widok po dacie ostatniej aktualizacji Poda

```
kubectl get pods --sort-by=.metadata.creationTimestamp
```

```
NAME                         READY   STATUS    RESTARTS   AGE
exercise-labels-first-pod    1/1     Running   0          10m
exercise-labels-second-pod   1/1     Running   0          10m
exercise-labels-third-pod    1/1     Running   0          10m
```

- Wybierz tylko i wyłączenie te Pody które nie są w fazie Running

```
kubectl get pods --field-selector status.phase!=Running
```

```
No resources found.
```