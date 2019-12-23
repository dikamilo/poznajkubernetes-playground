# Ćwiczenie

Stwórz CronJob tak aby:

1. co 2 minuty tworzył on Job

```yml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: exercise-job-schedule
spec:
  jobTemplate:
    metadata:
      name: exercise-job-schedule
    spec:
      template:
        spec:
          containers:
          - image: busybox
            name: exercise-job-schedule
            resources:
              limits:
                memory: "128Mi"
                cpu: "500m"
            command:
            - /bin/sh
            - -c
            - echo "job demo"
          restartPolicy: Never
  schedule: "*/2 * * * *"
```

```
k apply -f schedule.yml
```

```
NAME                    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
exercise-job-schedule   */2 * * * *   False     0        <none>          0s
exercise-job-schedule   */2 * * * *   False     1        4s              50s
exercise-job-schedule   */2 * * * *   False     0        14s             60s
```

```
NAME                               COMPLETIONS   DURATION   AGE
exercise-job-schedule-1577113080   0/1                      0s
exercise-job-schedule-1577113080   0/1           0s         0s
exercise-job-schedule-1577113080   1/1           6s         6s
```

```
NAME                                     READY   STATUS    RESTARTS   AGE
exercise-job-schedule-1577113080-sx252   0/1     Pending   0          0s
exercise-job-schedule-1577113080-sx252   0/1     Pending   0          0s
exercise-job-schedule-1577113080-sx252   0/1     ContainerCreating   0          0s
exercise-job-schedule-1577113080-sx252   0/1     Completed           0          6s
```

2. stworzony Job powinien tworzyć 2 lub więcej chodzące Pod

```yml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: exercise-job-multiple
spec:
  jobTemplate:
    metadata:
      name: exercise-job-multiple
    spec:
      parallelism: 2
      template:
        spec:
          containers:
          - image: busybox
            name: exercise-job-multiple
            resources:
              limits:
                memory: "128Mi"
                cpu: "500m"
            command:
            - /bin/sh
            - -c
            - echo "job demo"
          restartPolicy: Never
  schedule: "*/2 * * * *"
```

```
k apply -f multiple.yml
```

```
NAME                    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
exercise-job-multiple   */2 * * * *   False     0        <none>          0s
exercise-job-multiple   */2 * * * *   False     1        4s              56s
exercise-job-multiple   */2 * * * *   False     0        14s             66s
```

```
NAME                               COMPLETIONS   DURATION   AGE
exercise-job-multiple-1577113320   0/1 of 2                 0s
exercise-job-multiple-1577113320   0/1 of 2      0s         0s
exercise-job-multiple-1577113320   1/1 of 2      5s         5s
exercise-job-multiple-1577113320   2/1 of 2      9s         9s
```

```
NAME                                     READY   STATUS    RESTARTS   AGE
exercise-job-multiple-1577113320-bt8lt   0/1     Pending   0          0s
exercise-job-multiple-1577113320-9zf9p   0/1     Pending   0          0s
exercise-job-multiple-1577113320-bt8lt   0/1     Pending   0          0s
exercise-job-multiple-1577113320-9zf9p   0/1     Pending   0          0s
exercise-job-multiple-1577113320-bt8lt   0/1     ContainerCreating   0          0s
exercise-job-multiple-1577113320-9zf9p   0/1     ContainerCreating   0          0s
exercise-job-multiple-1577113320-9zf9p   0/1     Completed           0          5s
exercise-job-multiple-1577113320-bt8lt   0/1     Completed           0          9s
```

3. Pody powinny chodzić więcej niż 2 minuty. Możesz na stałe zaszyć 3 minuty 🙂

```yml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: exercise-job-long
spec:
  jobTemplate:
    metadata:
      name: exercise-job-long
    spec:
      parallelism: 2
      template:
        spec:
          containers:
          - image: busybox
            name: exercise-job-long
            resources:
              limits:
                memory: "128Mi"
                cpu: "500m"
            command:
            - /bin/sh
            - -c
            - sleep 3m
          restartPolicy: Never
  schedule: "*/2 * * * *"
```

```
k apply -f long.yml
```

```
NAME                SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
exercise-job-long   */2 * * * *   False     0        <none>          0s
exercise-job-long   */2 * * * *   False     1        5s              44s
exercise-job-long   */2 * * * *   False     2        5s              2m44s
exercise-job-long   */2 * * * *   False     1        75s             3m54s
```

```
NAME                           COMPLETIONS   DURATION   AGE
exercise-job-long-1577113680   0/1 of 2                 0s
exercise-job-long-1577113680   0/1 of 2      1s         1s
exercise-job-long-1577113800   0/1 of 2                 0s
exercise-job-long-1577113800   0/1 of 2      0s         0s
exercise-job-long-1577113680   1/1 of 2      3m5s       3m5s
exercise-job-long-1577113680   2/1 of 2      3m7s       3m7s
```

```
NAME                                 READY   STATUS    RESTARTS   AGE
exercise-job-long-1577113680-9s78w   0/1     Pending   0          0s
exercise-job-long-1577113680-9s78w   0/1     Pending   0          0s
exercise-job-long-1577113680-fgrgh   0/1     Pending   0          0s
exercise-job-long-1577113680-fgrgh   0/1     Pending   0          0s
exercise-job-long-1577113680-9s78w   0/1     ContainerCreating   0          0s
exercise-job-long-1577113680-fgrgh   0/1     ContainerCreating   0          0s
exercise-job-long-1577113680-9s78w   1/1     Running             0          4s
exercise-job-long-1577113680-fgrgh   1/1     Running             0          7s
exercise-job-long-1577113800-n5vjw   0/1     Pending             0          0s
exercise-job-long-1577113800-n5vjw   0/1     Pending             0          0s
exercise-job-long-1577113800-5bj6v   0/1     Pending             0          0s
exercise-job-long-1577113800-5bj6v   0/1     Pending             0          0s
exercise-job-long-1577113800-n5vjw   0/1     ContainerCreating   0          0s
exercise-job-long-1577113800-5bj6v   0/1     ContainerCreating   0          0s
exercise-job-long-1577113800-n5vjw   1/1     Running             0          5s
exercise-job-long-1577113800-5bj6v   1/1     Running             0          7s
exercise-job-long-1577113680-9s78w   0/1     Completed           0          3m4s
exercise-job-long-1577113680-fgrgh   0/1     Completed           0          3m6s
```

4. używając parametru concurrencyPolicy spróbuj uzyskać efekt, aby nowo utworzone Pod zastępowały stare, czyli nigdy żaden z Pod się nie zakończy

```yml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: exercise-job-replace
spec:
  jobTemplate:
    metadata:
      name: exercise-job-replace
    spec:
      parallelism: 2
      template:
        spec:
          containers:
          - image: busybox
            name: exercise-job-replace
            resources:
              limits:
                memory: "128Mi"
                cpu: "500m"
            command:
            - /bin/sh
            - -c
            - sleep 3m
          restartPolicy: Never
  schedule: "*/2 * * * *"
  concurrencyPolicy: Replace
```

```
k apply -f replace.yml
```

```
NAME                   SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
exercise-job-replace   */2 * * * *   False     0        <none>          0s
exercise-job-replace   */2 * * * *   False     1        5s              80s
exercise-job-replace   */2 * * * *   False     1        5s              3m20s
```

```
NAME                              COMPLETIONS   DURATION   AGE
exercise-job-replace-1577114160   0/1 of 2                 0s
exercise-job-replace-1577114160   0/1 of 2      0s         0s
exercise-job-replace-1577114160   0/1 of 2      2m         2m
exercise-job-replace-1577114280   0/1 of 2                 0s
exercise-job-replace-1577114280   0/1 of 2      0s         0s
```

```
NAME                                    READY   STATUS    RESTARTS   AGE
exercise-job-replace-1577114160-442pp   0/1     Pending   0          0s
exercise-job-replace-1577114160-9kv6d   0/1     Pending   0          0s
exercise-job-replace-1577114160-442pp   0/1     Pending   0          0s
exercise-job-replace-1577114160-9kv6d   0/1     Pending   0          0s
exercise-job-replace-1577114160-442pp   0/1     ContainerCreating   0          0s
exercise-job-replace-1577114160-9kv6d   0/1     ContainerCreating   0          0s
exercise-job-replace-1577114160-9kv6d   1/1     Running             0          5s
exercise-job-replace-1577114160-442pp   1/1     Running             0          7s
exercise-job-replace-1577114280-fxxpx   0/1     Pending             0          0s
exercise-job-replace-1577114280-fxxpx   0/1     Pending             0          0s
exercise-job-replace-1577114280-mffvz   0/1     Pending             0          0s
exercise-job-replace-1577114280-mffvz   0/1     Pending             0          0s
exercise-job-replace-1577114280-fxxpx   0/1     ContainerCreating   0          0s
exercise-job-replace-1577114280-mffvz   0/1     ContainerCreating   0          0s
exercise-job-replace-1577114160-442pp   1/1     Terminating         0          2m
exercise-job-replace-1577114160-9kv6d   1/1     Terminating         0          2m
exercise-job-replace-1577114280-mffvz   1/1     Running             0          5s
exercise-job-replace-1577114280-fxxpx   1/1     Running             0          8s
```