# Ćwiczenie

Korzystając z materiałów z lekcji przetestuj działanie zmiennych środowiskowych w praktyce.

1. Wykorzystaj proste zmienne środowiskowe.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-env
spec:
  containers:
  - name: exercise-env
    image: poznajkubernetes/pkad:blue
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    env:
      - name: PROJECT_NAME
        value: "Poznajkubernetes"
      - name: PROJECT_TYPE
        value: "K8s"
```

```
kubectl apply -f env.yml
```

```
kubectl port-forward env-exercise 8080
```

2. Wykorzystaj w `args` zmienne środowiskowe.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-args
spec:
  restartPolicy: Never
  containers:
  - name: exercise-args
    image: busybox
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    command:
      - echo
    args: 
      - "Hello from $(PROJECT_NAME)"
    env:
      - name: PROJECT_NAME
        value: "Poznajkubernetes"
```

```
kubectl apply -f args.yml
```

```
kubectl logs args-exercise
```

```
Hello from Poznajkubernetes
```

3. Skorzystaj z możliwości przekazania informacji o pod poprzez zmienne środowiskowe.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-pod-meta
spec:
  containers:
  - name: exercise-pod-meta
    image: poznajkubernetes/pkad:blue
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    env:
      - name: POD_NAME
        valueFrom:
          fieldRef:
            fieldPath: metadata.name
      - name: POD_IP
        valueFrom:
          fieldRef:
            fieldPath: status.podIP
```

```
kubectl apply -f pod-meta.yml
```

```
kubectl port-forward env-exercise 8080
```