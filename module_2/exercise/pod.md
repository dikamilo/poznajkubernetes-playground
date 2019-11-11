# Ćwiczenie 1

1. Utwórz definicje (plik YAML) pod z obrazem `busybox` za pomocą polecenia `kubectl run` z opcjami lub używając edytora tekstowego. Pamiętaj o opcjach `--dry-run` oraz `-o yaml`

```
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml >> pod.yml
```

2. Używając polecenia `kubectl create` stwórz Pod na podstawie definicji z punktu 1.

```
kubectl create -f pod.yml
```

3. Sprawdź status tworzenia Pod za pomocą `get`. Możesz też ustawić w osobnym oknie sprawdzanie „ciągłe” z opcją `-w` czyli `--watch` i zostawić je do końca ćwiczeń.

```
kubectl get pods -w
```

```
NAME      READY   STATUS    RESTARTS   AGE
busybox   0/1     Pending   0          0s
busybox   0/1     Pending   0          0s
busybox   0/1     ContainerCreating   0          1s
busybox   0/1     Completed           0          5s
```

4. Zastanów się dlaczego Pod przeszedł w stan `Completed`.

Pod zakończył swoją pracę.

# Ćwiczenie 2

1. Korzystając ze stanu klastra z ćwiczenia 1 oraz pliku YAML, przystąp do ćwiczenia.

2. Popraw plik YAML dodając `command` tak by Pod nie kończył od razu swojej pracy. Np: dodaj lub zmień tag w obrazie. Poprawne tag dla busybox znajdziesz na stronie: https://hub.docker.com/_/busybox

```yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - image: busybox:1.31
    name: busybox
    resources: {}
    command:
      - sleep
      - "60"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

3. Użyj polecenia kubectl create by wgrać nową definicję.

```
kubectl create -f pod.yml
```

```
Error from server (AlreadyExists): error when creating "ex2.yml": pods "busybox" already exists
```

4. Zastanów się dlaczego nie działa.

Pod z taką samą nazwą już istnieje, natomiast został utworzony bez zapisanej konfiguracji i K8s nie wie co się zmieniło w definicji Pod'a.

5. Usuń starą definicję za pomocą kubectl delete.

```
kubectl delete pod busybox
```

# Ćwiczenie 3

1. Utwórz w YAML definicję Pod, który nie kończy od razu swojej pracy. Możesz skorzystać z pliku stworzonego YAML w ćwiczeniu 2.

```yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - image: busybox:1.31
    name: busybox
    resources: {}
    command:
      - sleep
      - "60"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

2. Wgraj go na klaster używając polecenia `create`.

```
kubectl create -f pod.yml --save-config=true
```

3. Wykonaj modyfikację pliku np: zmień czas dla komendy `sleep`

```yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - image: busybox:1.31
    name: busybox
    resources: {}
    command:
      - sleep
      - "90"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

4. Spróbuj wgrać modyfikacje na klaster używając `create` lub `apply`.

```
kubectl apply -f pod.yml
```

```
The Pod "busybox" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)
```

5. Zastanów się i opisz na grupie kiedy warto używać `create`, a kiedy `apply`.

`apply` pozwala aplikować zmiany do "żyjących" obiektów, specyfikacja nie musi być pełna, ale ma też swoje ograniczenia co do pól które można zmieniać.

`create` nie pozwoli na utworzenie obiektu gdy ten już istnieje.

Można użyć `kubectl replace -f pod.yml --force` aby usunąć istniejący Pod i zastąpić go nowym.

# Ćwiczenie 4

1. Posprzątaj swój klaster tak by nie było na nim stworzonych przez Ciebie Pod.

```
kubectl delete pod busybox
```

2. Pamiętaj o get i delete.
3. Czy masz pomysł jak to zautomatyzować?

```
kubectl get pods | grep Completed | awk '{print $1}' | xargs kubectl delete pod
```

```
kubectl delete pod --field-selector=status.phase==Succeeded
```