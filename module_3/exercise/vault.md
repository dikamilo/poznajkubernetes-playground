# Ćwiczenie

Na podstawie demo i materiałów z poprzedniej lekcji wykorzystaj Vault do przekazania konfiguracji.

W swoim Pod wygeneruj plik konfiguracyjny typu json lub xml ze secretami z Vault.

Uruchomienie Vault w trybie developerskim w K8s:

```
kubectl apply -f pod-dev-vault.yml
```

```
kubectl port-forward dev-vault 8200 &
```

```
kubectl logs dev-vault | grep Token
```

```
export VAULT_TOKEN=s.dLJksnD8Z7kVsH2Ru7kIO34a
```

```
export VAULT_ADDR=http://127.0.0.1:8200
```

Konfiguracja szablonu generowania pliku:

```hcl
template {
  destination = "/etc/secrets/secrets.json"
  contents = <<EOH
  {
      {{- with secret "secret/myapp/config" }}
      "username": "{{ .Data.data.username }}",
      "password": "{{ .Data.data.password }}"
      {{ end }}
  }
  EOH
}
```

```
kubectl create cm exercise-vault-agent-config --from-file=./configs-k8s/
```

Konfiguracja autoryzacji K8s oraz polityk w Vault:

```bash
# Create the 'vault-auth' service account
kubectl apply --filename vault-auth-service-account.yml

# Create a policy named myapp-kv-ro
vault policy write myapp-kv-ro myapp-kv-ro.hcl

# Create test data in the `secret/myapp` path.
vault kv put secret/myapp/config username='appuser' password='my_password' ttl='30s'

# Set VAULT_SA_NAME to the service account you created earlier
export VAULT_SA_NAME=$(kubectl get sa vault-auth -o jsonpath="{.secrets[*]['name']}")

# Set SA_JWT_TOKEN value to the service account JWT used to access the TokenReview API
export SA_JWT_TOKEN=$(kubectl get secret $VAULT_SA_NAME -o jsonpath="{.data.token}" | base64 --decode; echo)

# Set SA_CA_CRT to the PEM encoded CA cert used to talk to Kubernetes API
export SA_CA_CRT=$(kubectl get secret $VAULT_SA_NAME -o jsonpath="{.data['ca\.crt']}" | base64 --decode; echo)

# Set K8S_HOST to minikube IP address
export K8S_HOST=kubernetes.docker.internal

# Enable the Kubernetes auth method at the default path ("auth/kubernetes")
vault auth enable kubernetes

# Tell Vault how to communicate with the Kubernetes (Minikube) cluster
vault write auth/kubernetes/config token_reviewer_jwt="$SA_JWT_TOKEN" kubernetes_host="https://$K8S_HOST:6443" kubernetes_ca_cert="$SA_CA_CRT"

# Create a role named, 'example' to map Kubernetes Service Account to
#  Vault policies and default token TTL
vault write auth/kubernetes/role/example bound_service_account_names=exercise-pod bound_service_account_namespaces=default policies=myapp-kv-ro ttl=24h
```

Definicja Pod:

```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  namespace: default
  name: exercise-pod
---
apiVersion: v1
kind: Pod
metadata:
  name: exercise-pod-vault-agent
spec:
  serviceAccountName: exercise-pod
  restartPolicy: Never
  volumes:
    - name: vault-token
      emptyDir:
        medium: Memory
    - name: config
      configMap:
        name: exercise-vault-agent-config
        items:
          - key: vault-agent-config.hcl
            path: vault-agent-config.hcl
          - key: consul-template-config.hcl
            path: consul-template-config.hcl
    - name: shared-data
      emptyDir: {}
  initContainers:
    - name: vault-agent-auth
      image: vault
      volumeMounts:
        - name: config
          mountPath: /etc/vault
        - name: vault-token
          mountPath: /home/vault
      env:
        - name: VAULT_ADDR
          value: http://10.1.0.94:8200
      args:
        [
          "agent",
          "-config=/etc/vault/vault-agent-config.hcl"
        ]

  containers:
    - name: consul-template
      image: hashicorp/consul-template:alpine
      imagePullPolicy: Always
      volumeMounts:
        - name: vault-token
          mountPath: /home/vault
        - name: config
          mountPath: /etc/consul-template
        - name: shared-data
          mountPath: /etc/secrets
      env:
        - name: HOME
          value: /home/vault
        - name: VAULT_ADDR
          value: http://10.1.0.94:8200
      args:
        [
          "-config=/etc/consul-template/consul-template-config.hcl"
        ]
    - name: exercise-secrets-from-vault
      image: poznajkubernetes/pkad:blue
      volumeMounts:
        - name: shared-data
          mountPath: /secrets
```

Wynik:

```
kubectl exec exercise-pod-vault-agent -c exercise-secrets-from-vault -- ls -la /secrets
```

```
total 12
drwxrwxrwx    2 root     root          4096 Nov 23 18:15 .
drwxr-xr-x    1 root     root          4096 Nov 23 18:16 ..
-rw-r--r--    1 100      1000            78 Nov 23 18:15 secrets.json
```

```
kubectl exec exercise-pod-vault-agent -c exercise-secrets-from-vault -- cat /secrets/secrets.json
```

```json
{
    "username": "appuser",
    "password": "my_password"
}
```