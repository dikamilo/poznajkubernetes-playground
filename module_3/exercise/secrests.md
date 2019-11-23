# Ćwiczenie 1 - stworzenie własnego prywatnego repozytorium kontenerów

Żebyś mógł skorzystać z prywatnego repozytorium kontenerów, musisz je wpierw mieć. Twoim pierwszym zadaniem jest utworzenie prywatnego repozytorium.

Do wybranego przez Ciebie repozytorium, wgraj własny obraz. 

```
docker login docker.pkg.github.com --username my_login -p my_token
```

```
docker images
```

```
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
poznajkubernetes/pkad                      blue                4305e828fdce        3 weeks ago         23MB
```

```
docker tag poznajkubernetes/pkad:blue docker.pkg.github.com/dikamilo/poznajkubernetes-playground/pkad:copy
```

```
docker push docker.pkg.github.com/dikamilo/poznajkubernetes-playground/pkad:copy
```

Aby K8s pobrał obraz z repo, usuwamy kopię lokalną:
```
docker rmi 4305e828fdce
```

https://github.com/dikamilo/poznajkubernetes-playground/packages/62259

# Ćwiczenie 2 - wykorzystanie obrazu z prywatnego repozytorium

Utwórz Pod, który ściągnie obraz z prywatnego repozytorium. Musisz wykonać następujące kroki:

1. Utworzyć secret typu `kubernetes.io/dockerconfigjson` na podstawie pliku `.docker/config.json`. W demo użyte było polecenie: `kubectl create secret generic regcred --from-file=.dockerconfigjson=/home/poznaj/.docker/con
fig.json --type=kubernetes.io/dockerconfigjson`

```
kubectl create secret generic githubcreg --from-file=.dockerconfigjson=/home/dikamilo/.docker/config.json --type=kubernetes.io/dockerconfigjson
```

```
kubectl get secrets githubcreg -o yaml
```

```yaml
apiVersion: v1
data:
  .dockerconfigjson: my-bash64-secret-here
kind: Secret
metadata:
  creationTimestamp: "2019-11-23T16:41:48Z"
  name: githubcreg
  namespace: default
  resourceVersion: "22567"
  selfLink: /api/v1/namespaces/default/secrets/githubcreg
  uid: 24dae63c-0e10-11ea-bb0c-00155dada007
type: kubernetes.io/dockerconfigjson
```

2. Użyć go w definicji Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-pod-secrets
spec:
  imagePullSecrets:
    - name: githubcreg
  containers:
  - name: exercise-pod-secrets
    image: docker.pkg.github.com/dikamilo/poznajkubernetes-playground/pkad:copy
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

3. Wgrać Pod do klastra.

```
kubectl apply -f pod-secrets.yml
```

```
kubectl describe pod exercise-pod-secrets
```

```
Events:
  Type    Reason     Age   From                     Message
  ----    ------     ----  ----                     -------
  Normal  Scheduled  8s    default-scheduler        Successfully assigned default/exercise-pod-secrets to docker-desktop
  Normal  Pulling    7s    kubelet, docker-desktop  Pulling image "docker.pkg.github.com/dikamilo/poznajkubernetes-playground/pkad:copy"
  Normal  Pulled     3s    kubelet, docker-desktop  Successfully pulled image "docker.pkg.github.com/dikamilo/poznajkubernetes-playground/pkad:copy"
  Normal  Created    3s    kubelet, docker-desktop  Created container exercise-pod-secrets
  Normal  Started    3s    kubelet, docker-desktop  Started container exercise-pod-secrets
```

# Ćwiczenie 3

Mając już obraz z prywatnego repozytorium, stwórz 2 rodzaje secret: `--from-literal` i `--from-file`, używając polecenia `kubectl create`. Gdy będziesz miał już je utworzone, spróbuj wykorzystać je jako pliki i/lub zmienne środowiskowe.

```
kubectl create secret generic exercise-secret-literal --from-literal=NAME=poznajkubernetes
```

```
echo URL=https://edu.poznajkubernetes.pl/ > exercise-secrets.txt
```

```
kubectl create secret generic exercise-secret-file --from-file=exercise-secrets.txt
```

```
kubectl get secrets
```

```
NAME                      TYPE                                  DATA   AGE
default-token-lvstf       kubernetes.io/service-account-token   3      2d21h
exercise-secret-file      Opaque                                1      4s
exercise-secret-literal   Opaque                                1      8s
githubcreg                kubernetes.io/dockerconfigjson        1      13m
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-pod-secrets-usage
spec:
  containers:
  - name: exercise-pod-secrets-usage
    image: docker.pkg.github.com/dikamilo/poznajkubernetes-playground/pkad:copy
    envFrom:
      - secretRef:
          name: exercise-secret-literal
    volumeMounts:
      - mountPath: /secrets/
        name: secret-vol
        readOnly: true
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
  volumes:
    - name: secret-vol
      secret:
          secretName: exercise-secret-file
```

```
kubectl apply -f pod-secrets-usage.yml
```

```
kubectl exec exercise-pod-secrets-usage -- printenv
```

```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=exercise-pod-secrets-usage
NAME=poznajkubernetes
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
HOME=/
```

```
kubectl exec exercise-pod-secrets-usage -- ls -la /secrets/
```

```
total 4
drwxrwxrwt    3 root     root           100 Nov 23 16:59 .
drwxr-xr-x    1 root     root          4096 Nov 23 16:59 ..
drwxr-xr-x    2 root     root            60 Nov 23 16:59 ..2019_11_23_16_59_44.810041411
lrwxrwxrwx    1 root     root            31 Nov 23 16:59 ..data -> ..2019_11_23_16_59_44.810041411
lrwxrwxrwx    1 root     root            27 Nov 23 16:59 exercise-secrets.txt -> ..data/exercise-secrets.txt
```