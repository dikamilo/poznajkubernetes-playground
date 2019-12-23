# Ćwiczenie

Korzystając z Ingress utworzonych przy okazji „Używanie reguł Ingress – Ćwiczenia” zmodyfikuje je o wykorzystanie certyfikatów.

W zależności od możliwości użyj self-sign lub Let’s Encrypt.

```
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
```

```
kubectl create secret tls tls-localhost --key key.pem --cert certificate.pem
```

```yml
apiVersion: v1
kind: Service
metadata:
  name: exercise-cert-echo-service
spec:
  selector:
    app: exercise-cert-echo
  ports:
  - port: 8080
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exercise-cert-echo
spec:
  selector:
    matchLabels:
      app: exercise-cert-echo
  template:
    metadata:
      labels:
        app: exercise-cert-echo
    spec:
      containers:
      - name: exercise-cert-echo
        image: gcr.io/google_containers/echoserver:1.4
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: exercise-cert
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  tls:
    - hosts:
      - localhost
      secretName: tls-localhost
  rules:
    - host: localhost
      http:
        paths:
          - backend:
              serviceName: exercise-cert-echo-service
              servicePort: 8080
```

```
k apply -f ingress.yml
```

```
curl --insecure -v https://localhost/
```

```
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Client hello (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
*  start date: Dec 23 12:41:04 2019 GMT
*  expire date: Dec 22 12:41:04 2020 GMT
*  issuer: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x7fffcb160580)
> GET / HTTP/2
> Host: localhost
> User-Agent: curl/7.58.0
> Accept: */*
>
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
< HTTP/2 200
< server: openresty/1.15.8.2
< date: Mon, 23 Dec 2019 13:35:30 GMT
< content-type: text/plain
< vary: Accept-Encoding
< strict-transport-security: max-age=15724800; includeSubDomains
<
CLIENT VALUES:
client_address=10.1.0.225
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://localhost:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=localhost
user-agent=curl/7.58.0
x-forwarded-for=192.168.65.3
x-forwarded-host=localhost
x-forwarded-port=443
x-forwarded-proto=https
x-real-ip=192.168.65.3
x-request-id=b6c6ecc160d392075046893523e6eef3
x-scheme=https
BODY:
* Connection #0 to host localhost left intact
-no body in request-
```