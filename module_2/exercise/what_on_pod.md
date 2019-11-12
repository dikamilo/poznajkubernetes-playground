# Ćwiczenie 1

1. Stwórz Pod na bazie obrazu z modułu 1

```yml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
  - name: kuard
    image: poznajkubernetes/kuard
```

```
kubectl create -f kuard.yml
```

2. Zobacz w jakim stanie on się znajduje

```
kubectl get pod
```

```
NAME       READY   STATUS    RESTARTS   AGE
kuard   1/1     Running   0          15s
```

3. Podejrzyj jego logi

```
kubectl logs kuard
```

```
2019/11/12 16:55:58 Starting pkad version: blue
2019/11/12 16:55:58 **********************************************************************
2019/11/12 16:55:58 * WARNING: This server may expose sensitive
2019/11/12 16:55:58 * and secret information. Be careful.
2019/11/12 16:55:58 **********************************************************************
2019/11/12 16:55:58 Config:
{
  "address": ":8080",
  "debug": false,
  "debug-sitedata-dir": "./sitedata",
  "keygen": {
    "enable": false,
    "exit-code": 0,
    "exit-on-complete": false,
    "memq-queue": "",
    "memq-server": "",
    "num-to-gen": 0,
    "time-to-run": 0
  },
  "liveness": {
    "fail-next": 0
  },
  "readiness": {
    "fail-next": 0
  },
  "tls-address": ":8443",
  "tls-dir": "/tls"
}
2019/11/12 16:55:58 Could not find certificates to serve TLS
2019/11/12 16:55:58 Serving on HTTP on :8080
```

4. Wykonaj w koneterze wylistowanie katalogów

```
kubectl exec kuard -- ls -la
```

```
total 17076
drwxr-xr-x    1 root     root          4096 Nov 12 16:55 .
drwxr-xr-x    1 root     root          4096 Nov 12 16:55 ..
-rwxr-xr-x    1 root     root             0 Nov 12 16:55 .dockerenv
drwxr-xr-x    2 root     root          4096 Aug 20 10:30 bin
drwxr-xr-x    5 root     root           360 Nov 12 16:55 dev
drwxr-xr-x    1 root     root          4096 Nov 12 16:55 etc
drwxr-xr-x    2 root     root          4096 Aug 20 10:30 home
drwxr-xr-x    5 root     root          4096 Aug 20 10:30 lib
drwxr-xr-x    5 root     root          4096 Aug 20 10:30 media
drwxr-xr-x    2 root     root          4096 Aug 20 10:30 mnt
drwxr-xr-x    2 root     root          4096 Aug 20 10:30 opt
-rwxr-xr-x    1 root     root      17419358 Oct 28 12:03 pkad
dr-xr-xr-x  239 root     root             0 Nov 12 16:55 proc
drwx------    2 root     root          4096 Aug 20 10:30 root
drwxr-xr-x    1 root     root          4096 Nov 12 16:55 run
drwxr-xr-x    2 root     root          4096 Aug 20 10:30 sbin
drwxr-xr-x    2 root     root          4096 Aug 20 10:30 srv
dr-xr-xr-x   13 root     root             0 Nov 12 16:55 sys
drwxrwxrwt    2 root     root          4096 Aug 20 10:30 tmp
drwxr-xr-x    7 root     root          4096 Aug 20 10:30 usr
drwxr-xr-x   11 root     root          4096 Aug 20 10:30 var
```

5. Odpytaj się http://localhost:PORT w kontenerze

```
kubectl exec kuard -- wget -qO- localhost:8080
```

```html
<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8">

  <title>Poznaj Kubernetes Demo</title>

  <link rel="stylesheet" href="/static/css/bootstrap.min.css">
  <link rel="stylesheet" href="/static/css/styles.css">

  <script>
var pageContext = {"urlBase":"","hostname":"kuard","addrs":["10.1.0.57"],"version":"blue","versionColor":"hsl(21,100%,50%)","requestDump":"GET / HTTP/1.1\r\nHost: localhost:8080\r\nConnection: close\r\nConnection: close\r\nUser-Agent: Wget","requestProto":"HTTP/1.1","requestAddr":"127.0.0.1:40666"}
  </script>
</head>


<svg style="position: absolute; width: 0; height: 0; overflow: hidden;" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<defs>
<symbol id="icon-power" viewBox="0 0 32 32">
<title>power</title>
<path class="path1" d="M12 0l-12 16h12l-8 16 28-20h-16l12-12z"></path>
</symbol>
<symbol id="icon-notification" viewBox="0 0 32 32">
<title>notification</title>
<path class="path1" d="M16 3c-3.472 0-6.737 1.352-9.192 3.808s-3.808 5.72-3.808 9.192c0 3.472 1.352 6.737 3.808 9.192s5.72 3.808 9.192 3.808c3.472 0 6.737-1.352 9.192-3.808s3.808-5.72 3.808-9.192c0-3.472-1.352-6.737-3.808-9.192s-5.72-3.808-9.192-3.808zM16 0v0c8.837 0 16 7.163 16 16s-7.163 16-16 16c-8.837 0-16-7.163-16-16s7.163-16 16-16zM14 22h4v4h-4zM14 6h4v12h-4z"></path>
</symbol>
</defs>
</svg>

<body>
  <div id="root"></div>
  <script src="/built/bundle.js" type="text/javascript"></script>
  <script src="/static/js/jquery-3.3.1.slim.min.js" type="text/javascript"></script>
  <script src="/static/js/popper.min.js" type="text/javascript"></script>
  <script src="/static/js/bootstrap.min.js" type="text/javascript"></script>
</body>
</html>
```

6. Zrób to samo z powołanego osobno poda (kubectl run)

```
kubectl describe pod kuard
```

Klastrowe IP Pod: 10.1.0.57 

```
kubectl run bb --image=busybox --restart=Never --rm -it -- wget -qO- 10.1.0.57:8080
```

```html
<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8">

  <title>Poznaj Kubernetes Demo</title>

  <link rel="stylesheet" href="/static/css/bootstrap.min.css">
  <link rel="stylesheet" href="/static/css/styles.css">

  <script>
var pageContext = {"urlBase":"","hostname":"kuard","addrs":["10.1.0.57"],"version":"blue","versionColor":"hsl(21,100%,50%)","requestDump":"GET / HTTP/1.1\r\nHost: 10.1.0.57:8080\r\nConnection: close\r\nConnection: close\r\nUser-Agent: Wget","requestProto":"HTTP/1.1","requestAddr":"10.1.0.59:60954"}
  </script>
</head>


<svg style="position: absolute; width: 0; height: 0; overflow: hidden;" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<defs>
<symbol id="icon-power" viewBox="0 0 32 32">
<title>power</title>
<path class="path1" d="M12 0l-12 16h12l-8 16 28-20h-16l12-12z"></path>
</symbol>
<symbol id="icon-notification" viewBox="0 0 32 32">
<title>notification</title>
<path class="path1" d="M16 3c-3.472 0-6.737 1.352-9.192 3.808s-3.808 5.72-3.808 9.192c0 3.472 1.352 6.737 3.808 9.192s5.72 3.808 9.192 3.808c3.472 0 6.737-1.352 9.192-3.808s3.808-5.72 3.808-9.192c0-3.472-1.352-6.737-3.808-9.192s-5.72-3.808-9.192-3.808zM16 0v0c8.837 0 16 7.163 16 16s-7.163 16-16 16c-8.837 0-16-7.163-16-16s7.163-16 16-16zM14 22h4v4h-4zM14 6h4v12h-4z"></path>
</symbol>
</defs>
</svg>

<body>
  <div id="root"></div>
  <script src="/built/bundle.js" type="text/javascript"></script>
  <script src="/static/js/jquery-3.3.1.slim.min.js" type="text/javascript"></script>
  <script src="/static/js/popper.min.js" type="text/javascript"></script>
  <script src="/static/js/bootstrap.min.js" type="text/javascript"></script>
</body>
</html>
pod "bb" deleted
```

7. Dostań się do poda za pomocą przekierowania portów

```
kubectl port-forward kuard 8080:8080
```

```
wget -qO- localhost:8080
```

```html
<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8">

  <title>Poznaj Kubernetes Demo</title>

  <link rel="stylesheet" href="/static/css/bootstrap.min.css">
  <link rel="stylesheet" href="/static/css/styles.css">

  <script>
var pageContext = {"urlBase":"","hostname":"kuard","addrs":["10.1.0.57"],"version":"blue","versionColor":"hsl(21,100%,50%)","requestDump":"GET / HTTP/1.1\r\nHost: localhost:8080\r\nAccept: */*\r\nAccept-Encoding: identity\r\nConnection: Keep-Alive\r\nUser-Agent: Wget/1.19.4 (linux-gnu)","requestProto":"HTTP/1.1","requestAddr":"127.0.0.1:39172"}
  </script>
</head>


<svg style="position: absolute; width: 0; height: 0; overflow: hidden;" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<defs>
<symbol id="icon-power" viewBox="0 0 32 32">
<title>power</title>
<path class="path1" d="M12 0l-12 16h12l-8 16 28-20h-16l12-12z"></path>
</symbol>
<symbol id="icon-notification" viewBox="0 0 32 32">
<title>notification</title>
<path class="path1" d="M16 3c-3.472 0-6.737 1.352-9.192 3.808s-3.808 5.72-3.808 9.192c0 3.472 1.352 6.737 3.808 9.192s5.72 3.808 9.192 3.808c3.472 0 6.737-1.352 9.192-3.808s3.808-5.72 3.808-9.192c0-3.472-1.352-6.737-3.808-9.192s-5.72-3.808-9.192-3.808zM16 0v0c8.837 0 16 7.163 16 16s-7.163 16-16 16c-8.837 0-16-7.163-16-16s7.163-16 16-16zM14 22h4v4h-4zM14 6h4v12h-4z"></path>
</symbol>
</defs>
</svg>

<body>
  <div id="root"></div>
  <script src="/built/bundle.js" type="text/javascript"></script>
  <script src="/static/js/jquery-3.3.1.slim.min.js" type="text/javascript"></script>
  <script src="/static/js/popper.min.js" type="text/javascript"></script>
  <script src="/static/js/bootstrap.min.js" type="text/javascript"></script>
</body>
</html>
```

8. Dostań się do poda za pomocą API Server

```
kubectl proxy
```

```
Starting to serve on 127.0.0.1:8001
```

```
wget -qO- http://127.0.0.1:8001/api/v1/namespaces/default/pods/http:kuard:8080/proxy/
```

```html
<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8">

  <title>Poznaj Kubernetes Demo</title>

  <link rel="stylesheet" href="/api/v1/namespaces/default/pods/http:kuard:8080/proxy/static/css/bootstrap.min.css">       <link rel="stylesheet" href="/api/v1/namespaces/default/pods/http:kuard:8080/proxy/static/css/styles.css">

  <script>
var pageContext = {"urlBase":"","hostname":"kuard","addrs":["10.1.0.57"],"version":"blue","versionColor":"hsl(21,100%,50%)","requestDump":"GET / HTTP/1.1\r\nHost: 127.0.0.1:8001\r\nAccept: */*\r\nAccept-Encoding: identity\r\nUser-Agent: Wget/1.19.4 (linux-gnu)\r\nX-Forwarded-For: 127.0.0.1, 127.0.0.1\r\nX-Forwarded-Uri: /api/v1/namespaces/default/pods/http:kuard:8080/proxy/","requestProto":"HTTP/1.1","requestAddr":"10.1.0.1:59660"}
  </script>
</head>


<svg style="position: absolute; width: 0; height: 0; overflow: hidden;" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<defs>
<symbol id="icon-power" viewbox="0 0 32 32">
<title>power</title>
<path class="path1" d="M12 0l-12 16h12l-8 16 28-20h-16l12-12z"></path>
</symbol>
<symbol id="icon-notification" viewbox="0 0 32 32">
<title>notification</title>
<path class="path1" d="M16 3c-3.472 0-6.737 1.352-9.192 3.808s-3.808 5.72-3.808 9.192c0 3.472 1.352 6.737 3.808 9.192s5.72 3.808 9.192 3.808c3.472 0 6.737-1.352 9.192-3.808s3.808-5.72 3.808-9.192c0-3.472-1.352-6.737-3.808-9.192s-5.72-3.808-9.192-3.808zM16 0v0c8.837 0 16 7.163 16 16s-7.163 16-16 16c-8.837 0-16-7.163-16-16s7.163-16 16-16zM14 22h4v4h-4zM14 6h4v12h-4z"></path>
</symbol>
</defs>
</svg>

<body>
  <div id="root"></div>
  <script src="/api/v1/namespaces/default/pods/http:kuard:8080/proxy/built/bundle.js" type="text/javascript"></script>    <script src="/api/v1/namespaces/default/pods/http:kuard:8080/proxy/static/js/jquery-3.3.1.slim.min.js" type="text/javascript"></script>
  <script src="/api/v1/namespaces/default/pods/http:kuard:8080/proxy/static/js/popper.min.js" type="text/javascript"></script>
  <script src="/api/v1/namespaces/default/pods/http:kuard:8080/proxy/static/js/bootstrap.min.js" type="text/javascript"></script>
</body>
</html>
```



# Ćwiczenie 2

1. Stwórz Pod zawierający dwa kontenery – busybox i poznajkubernetes/helloapp:multi

```yml
apiVersion: v1
kind: Pod
metadata:
 name: two-containers
spec:
 containers:
 - image: poznajkubernetes/helloapp:multi
   name: helloapp
 - image: busybox
   name: busybox
```

```
kubectl create -f two-containers.yml
```

2. Zweryfikuj, że Pod działa poprawnie

```
kubectl get pod
```

```
NAME             READY   STATUS             RESTARTS   AGE
two-containers   1/2     CrashLoopBackOff   1          20s
```

3. Jak nie działa, dowiedz się dlaczego

```
kubectl describe pod two-containers
```

```
Containers:
  helloapp:
    State:          Running
      Started:      Tue, 12 Nov 2019 18:10:00 +0100
  busybox:
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 12 Nov 2019 18:10:55 +0100
      Finished:     Tue, 12 Nov 2019 18:10:55 +0100
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 12 Nov 2019 18:10:26 +0100
      Finished:     Tue, 12 Nov 2019 18:10:26 +0100
    Ready:          False
    Restart Count:  3
```

```
Events:
  Type     Reason     Age               From                     Message
  ----     ------     ----              ----                     -------
  Normal   Scheduled  61s               default-scheduler        Successfully assigned default/two-containers to docker-desktop
  Normal   Pulling    60s               kubelet, docker-desktop  Pulling image "poznajkubernetes/helloapp:multi"          Normal   Pulled     56s               kubelet, docker-desktop  Successfully pulled image "poznajkubernetes/helloapp:multi"
  Normal   Created    56s               kubelet, docker-desktop  Created container helloapp
  Normal   Started    56s               kubelet, docker-desktop  Started container helloapp
  Normal   Pulling    5s (x4 over 56s)  kubelet, docker-desktop  Pulling image "busybox"
  Normal   Pulled     1s (x4 over 52s)  kubelet, docker-desktop  Successfully pulled image "busybox"
  Normal   Created    1s (x4 over 52s)  kubelet, docker-desktop  Created container busybox
  Normal   Started    1s (x4 over 52s)  kubelet, docker-desktop  Started container busybox
  Warning  BackOff    0s (x5 over 47s)  kubelet, docker-desktop  Back-off restarting failed container
```

4. Zastanów się nad rozwiązaniem problemu jeżeli istnieje – co można by było zrobić i jak

- zmienić `command` w kontenerze busybox aby kontener nie konczył od razu swojej pracy i pracował "ciągle"
- jeżeli kontener busybox ma zrobić zadanie i się zakończyć to przenieść go do `initContainers`
- ustawić `restartPolicy` na `OnFailure` co spowoduje:

```
kubectl get pod -w
```

```
NAME             READY   STATUS    RESTARTS   AGE
two-containers   0/2     Pending   0          0s
two-containers   0/2     Pending   0          0s
two-containers   0/2     ContainerCreating   0          0s
two-containers   1/2     Running             0          6s
```