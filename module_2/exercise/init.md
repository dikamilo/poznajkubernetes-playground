# Ćwiczenie

Celem ćwiczenia jest pobranie strony www z repozytorium git za pomocą kontenera typu init. Pobrane dane muszą trafić na wolumen typu `emptyDir` i zostać wykorzystane do serwowania treści w głównym kontenerze.

Do kontenera init z git zbuduj obraz na bazie ubuntu. Należy doinstalować `git`.
Do kontenera serwującego treść wykorzystaj `nginx`.

```Dockerfile
FROM ubuntu

RUN apt-get update && apt-get install -y git
```

```
docker login docker.pkg.github.com --username my_login -p my_token
docker build -t docker.pkg.github.com/dikamilo/poznajkubernetes-playground/exercise:init-container -f Dockerfile.download-from-git .
docker push docker.pkg.github.com/dikamilo/poznajkubernetes-playground/exercise:init-container
```

```yml
apiVersion: v1
kind: Pod
metadata:
  name: init-exercise
spec:
  containers:
  - name: init-exercise
    image: nginx
    ports:
      - containerPort: 80
    volumeMounts:
      - name: workdir
        mountPath: /usr/share/nginx/html
  initContainers:
    - name: download-from-git
      image: docker.pkg.github.com/dikamilo/poznajkubernetes-playground/exercise:init-container
      imagePullPolicy: IfNotPresent
      command:
        - git
        - clone
        - "https://github.com/PoznajKubernetes/poznajkubernetes.github.io"
        - /work-dir
      volumeMounts:
        - name: workdir
          mountPath: /work-dir
  volumes:
    - name: workdir
      emptyDir: {}
```

```
kubectl apply -f init-exercise.yml
```

```
kubectl port-forward init-exercise 8080:80
```