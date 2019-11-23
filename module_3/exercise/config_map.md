# Ćwiczenie 1 - Praca z ConfigMap

1. Stwórz ConfigMap wykorzystując `kubectl`.

- Załącz do niej przynajmniej dwie proste wartości (literal)
- Załącz do niej klucz: `123_TESTING` z dowolną wartością
- Załącz do niej klucz: `TESTING-123` z dowolną wartością
- Załącz do niej klucz: `TESTING` z dowolną wartością

```
kubectl create cm first-cm --from-literal=123_TESTING=first_value --from-literal=TESTING-123=second_value --from-literal=TESTING=third_value
```

```
kubectl describe cm first-cm
```

```
Name:         first-cm
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
123_TESTING:
----
first_value
TESTING:
----
third_value
TESTING-123:
----
second_value
Events:  <none>
```

2. Stwórz drugą ConfigMap wykorzystując `kubectl`.

- Załącz do niej dwie takie same klucze i ale różne wartości.
- Jeden plik normalnie.
- Oraz jeden plik z inną nazwą klucza niż nazwa pliku.

```
kubectl create cm second-cm --from-literal=TESTING=first_value --from-literal=TESTING=second_value --from-file=config.toml --from-file=my_json_config=config.json
```

```
error: cannot add key TESTING, another key by that name already exists in data: map[TESTING:first_value config.toml:[owner]
```

3. Stwórz trzecią ostatnią ConfigMapę wykorzystując `kubectl`.

- zrób, tak by załączyć pliki o rozmiarach ~20KB, ~30KB, ~40KB i ~50KB.

```
perl -e 'print "2" x 20000' > 20k.txt
perl -e 'print "3" x 30000' > 30k.txt
perl -e 'print "4" x 40000' > 40k.txt
perl -e 'print "5" x 20000' > 50k.txt
```

```
kubectl create cm third-cm --from-file=20k.txt --from-file=30k.txt  --from-file=40k.txt  --from-file=50k.txt 
```

Wyeksportuj wszystkie stworzone ConfigMapy do yamli.

```
kubectl create cm first-cm --from-literal=123_TESTING=first_value --from-literal=TESTING-123=second_value --from-literal=TESTING=third_value --dry-run=true -o yaml > first-cm.yml
```

```
kubectl create cm second-cm --from-literal=TESTING=first_value --from-literal=TESTING_2=second_value --from-file=config.toml --from-file=my_json_config=config.json --dry-run=true -o yaml > second-cm.yml
```

```
kubectl create cm third-cm --from-file=20k.txt --from-file=30k.txt  --from-file=40k.txt  --from-file=50k.txt --dry-run=true -o yaml > third-cm.yml
```

Odpowiedz sobie na pytania:

1. Co się stanie, gdy nadamy taki sam klucz? Czego Ty byś się spodziewał?

Dostaniemy błąd z informacją, że klucz z taką nazwą już istnieje. Nazwy kluczy w config mapie muszą być unikalne.

2. Czy można nadać dowolną nazwę klucza w ConfigMap?

Nazwa klucza jest sprawdzana następującym wyrażeniem regexp: `[-._a-zA-Z0-9]+`


# Ćwiczenie 2 - Zmienne środowiskowe

Będziemy korzystać z naszych ConfigMap z poprzedniej sekcji. Ćwiczenia te polegają na obserwowaniu wyniku akcji i zastanowieniu się, dlaczego wynik jest taki, a nie inny.

1. Wczytaj wszystkie klucze z **pierwszej** ConfigMapy do Poda jako zmienne środowiskowe. Zweryfikuj dokładnie zmienne środowiskowe. Jaki wynik został uzyskany i dlaczego taki?

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-pod-first-cm-env
spec:
  containers:
  - name: exercise-pod-first-cm-env
    image: poznajkubernetes/pkad:blue
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    envFrom:
      - configMapRef:
          name: first-cm
```

```
kubectl apply -f pod-first-cm-env.yml
```

```
kubectl exec exercise-pod-first-cm-env -- printenv
```

```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=exercise-pod-meta
TESTING-123=second_value
TESTING=third_value
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
HOME=/
```

W Pod są widoczne wartości `TESTING` oraz `TESTING-123`. Natomiast wartość `123_TESTING` nie jest widoczna:

```
kubectl describe pod exercise-pod-first-cm-env
```

```
Warning  InvalidEnvironmentVariableNames  8m57s  kubelet, docker-desktop  Keys [123_TESTING] from the EnvFrom configMap default/first-cm were skipped since they are considered invalid environment variable names.
```

2. Wczytaj wszystkie klucze z **trzeciej** ConfigMapy do Poda jako zmienne środowiskowe. Zweryfikuj dokładnie zmienne środowiskowe. Jaki wynik został uzyskany i dlaczego taki?

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-pod-third-cm-env
spec:
  containers:
  - name: exercise-pod-third-cm-env
    image: poznajkubernetes/pkad:blue
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    envFrom:
      - configMapRef:
          name: third-cm
```

```
kubectl apply -f pod-third-cm-env.yml
```

```
kubectl exec exercise-pod-third-cm-env -- printenv
```

```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=exercise-pod-third-cm-env
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
HOME=/
```

Wartości z config mapy nie są widoczne w Pod. Nazwy kluczy nie są poprawne. Zmienne środowiskowe są weryfikowane następującym wyrażeniem regexp: `[a-zA-Z_]+[a-zA-Z0-9_]*`

```
kubectl describe pod exercise-pod-third-cm-env
```

```
Warning  InvalidEnvironmentVariableNames  45s   kubelet, docker-desktop  Keys [20k.txt, 30k.txt, 40k.txt, 50k.txt] from the EnvFrom configMap default/third-cm were skipped since they are considered invalid environment variable names.
```

W przypadku zmiany kluczy w config mapie na poprawne np. zamiast nazw plików następujące nazwy: `FILE_20K`, `FILE_30K`, `FILE_40K`, `FILE_50K` - wartości są widoczne w Pod.


Odpowiedz sobie na pytania:

1. Co ma pierwszeństwo: zmienna środowiskowa zdefiniowana w ConfigMap czy w Pod?

Pierwszeństwo ma zmienna w Pod. 

2. Czy kolejność definiowania ma znaczenie (np.: env przed envFrom)?

Zmienne są wczytywane w kolejności definiowania w Pod od góry do dołu i jeżeli występują duplikaty nazw, to wartość zostanie nadpisana na ostatnią wartość dla tej nazwy.

3. Jak się ma kolejność do dwóch różnych ConfigMap?

Jeżeli wczytywane są dwa różne ConfigMap oraz zawierają między sobą takie same nazwy kluczy, wartość klucza zostanie ustawiona na ostatnią wczytaną wartość:

```
kubectl create cm first-cm --from-literal=TESTING=first_value
```

```
kubectl create cm second-cm --from-literal=TESTING=second_value
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-multiple-cm
spec:
  containers:
  - name: pod-multiple-cm
    image: poznajkubernetes/pkad:blue
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    envFrom:
      - configMapRef:
          name: first-cm
      - configMapRef:
          name: second-cm
```

```
kubectl exec pod-multiple-cm -- printenv
```

```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=test-pod
TESTING=second_value
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
HOME=/
```

# Ćwiczenie 3 - Wolumeny

1. Wykorzystując drugą ConfigMapę, stwórz Pod i wczytaj wszystkie pliki do katalogu wybranego przez siebie katalogu.


Utworzono nową ConfigMapę `files-cm` tylko z plikami, ponieważ ta z poprzedniego ćwiczenia zwraca błąd przy tworzeniu z duplikacją wartości.

```
kubectl create cm files-cm --from-file=config.toml --from-file=my_json_config=config.json
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-pod-files-cm-volume
spec:
  volumes:
    - name: files-cm-volume
      configMap:
          name: files-cm
  containers:
  - name: exercise-pod-files-cm-volume
    image: poznajkubernetes/pkad:blue
    volumeMounts:
      - mountPath: /config
        name: files-cm-volume
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

```
kubectl apply -f pod-files-cm-volume.yml
```

```
kubectl exec exercise-pod-files-cm-volume -- ls -la /config
```

```
total 12
drwxrwxrwx    3 root     root          4096 Nov 20 21:05 .
drwxr-xr-x    1 root     root          4096 Nov 20 21:05 ..
drwxr-xr-x    2 root     root          4096 Nov 20 21:05 ..2019_11_20_21_05_21.190493222
lrwxrwxrwx    1 root     root            31 Nov 20 21:05 ..data -> ..2019_11_20_21_05_21.190493222
lrwxrwxrwx    1 root     root            18 Nov 20 21:05 config.toml -> ..data/config.toml
lrwxrwxrwx    1 root     root            21 Nov 20 21:05 my_json_config -> ..data/my_json_config
```

```
kubectl exec exercise-pod-files-cm-volume -- cat /config/my_json_config
```

```
{
    "section": {
        "key_one": "first value",
        "key_two": "second value"
    }
}
```

2. Wczytaj do wolumenu tylko i wyłącznie pliki powyżej 30 KB z trzeciej ConfigMapy.


```
kubectl create cm third-cm --from-file=20k.txt --from-file=30k.txt  --from-file=40k.txt  --from-file=50k.txt 
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exercise-pod-third-cm-volume
spec:
  volumes:
    - name: third-cm-volume
      configMap:
          name: third-cm
          items:
            - key: 40k.txt
              path: 40k.txt
            - key: 50k.txt
              path: 50k.txt
  containers:
  - name: exercise-pod-third-cm-volume
    image: poznajkubernetes/pkad:blue
    volumeMounts:
      - mountPath: /config
        name: third-cm-volume
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```


```
kubectl apply -f pod-third-cm-volume.yml
```

```
kubectl exec exercise-pod-third-cm-volume -- ls -la /config
```

```
total 12
drwxrwxrwx    3 root     root          4096 Nov 20 21:11 .
drwxr-xr-x    1 root     root          4096 Nov 20 21:11 ..
drwxr-xr-x    2 root     root          4096 Nov 20 21:11 ..2019_11_20_21_11_08.500706490
lrwxrwxrwx    1 root     root            31 Nov 20 21:11 ..data -> ..2019_11_20_21_11_08.500706490
lrwxrwxrwx    1 root     root            14 Nov 20 21:11 40k.txt -> ..data/40k.txt
lrwxrwxrwx    1 root     root            14 Nov 20 21:11 50k.txt -> ..data/50k.txt
```

Odpowiedz sobie na pytania:

1. Co się stanie jak z `mountPath` ustawisz na katalog Twojej aplikacji?

Katalog aplikacji zostanie nadpisany tym co jest ustawione w `mountPath`.

2. Co się stanie, jak plik stworzony przez ConfigMap zostanie usunięty? Czy taki plik zostanie usunięty? co spowoduje aktualizacja ConfigMapy?

Pliki z ConfigMapy są montowane w trybie read-only, nie da się ich usunąć w Pod:

```
kubectl exec exercise-pod-third-cm-volume -- rm /config/50k.txt
```

```
rm: can't remove '/config/50k.txt': Read-only file system
command terminated with exit code 1
```

Jeżeli ConfigMapa zostanie usunięta, w Pod nic się nie zmieni.

Aktualizacja ConfigMapy spowoduje, że zawartość pliku w Pod się zmieni na zaktualizowane dane.