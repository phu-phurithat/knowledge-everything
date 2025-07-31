# Setup for K8s cluster

## Step-by-step: Use Vault Secrets in Kubernetes Apps

***

### Step 1: **Install and Run Vault**

* Deploy Vault server (can be on Kubernetes or external).
* Initialize and unseal Vault.
* Login as admin.

***

### Step 2: **Enable Kubernetes Auth method**

```bash
vault auth enable kubernetes
```

***

### Step 3: **Configure Kubernetes Auth method**

You need to provide Vault with:

* Kubernetes API server URL
* Kubernetes CA cert
* JWT token reviewer ServiceAccount token

To get those:

```bash
# From inside Kubernetes cluster (or using kubectl proxy)
KUBE_HOST=$(kubectl config view --raw -o jsonpath='{.clusters[0].cluster.server}')
KUBE_CA_CERT=$(kubectl get secret $(kubectl get serviceaccount default -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.ca\.crt}' | base64 --decode)
KUBE_TOKEN_REVIEWER_JWT=$(kubectl get secret $(kubectl get serviceaccount default -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 --decode)
```

Configure Vault:

```bash
vault write auth/kubernetes/config \
    token_reviewer_jwt="$KUBE_TOKEN_REVIEWER_JWT" \
    kubernetes_host="$KUBE_HOST" \
    kubernetes_ca_cert="$KUBE_CA_CERT"
```

***

### Step 4: **Create Vault policy for your app**

Create a policy file `my-app-policy.hcl`:

<pre class="language-hcl"><code class="lang-hcl"><strong>path "kv/data/my-app/*" {
</strong>  capabilities = ["read"]
}
</code></pre>

Apply policy:

```bash
vault policy write my-app-policy my-app-policy.hcl
```

***

### Step 5: **Create Kubernetes Auth role**

Map K8s ServiceAccount(s) to Vault policy:

```bash
vault write auth/kubernetes/role/my-app-role \
    bound_service_account_names=my-app-sa \
    bound_service_account_namespaces=default \
    policies=my-app-policy \
    ttl=24h
```

***

### Step 6: **Store secrets in Vault**

```bash
vault kv put kv/my-app/config username="dbuser" password="dbpass123"
```

***

### Step 7: **Create a Kubernetes ServiceAccount for your app**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
```

Apply:

```bash
kubectl apply -f serviceaccount.yaml
```

***

### Step 8: **Deploy your app with Vault Agent sidecar**

Example deployment snippet with Vault Agent injector annotations:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "my-app-role"
        vault.hashicorp.com/agent-inject-secret-config.txt: "kv/data/my-app/config"
        vault.hashicorp.com/agent-inject-template-config.txt: |
          {{- with secret "kv/data/my-app/config" -}}
          export USERNAME={{ .Data.data.username }}
          export PASSWORD={{ .Data.data.password }}
          {{- end }}
    spec:
      serviceAccountName: my-app-sa
      containers:
      - name: app
        image: busybox
        command: ["/bin/sh", "-c"]
        args:
          - |
            source /vault/secrets/config.txt
            echo "DB USER: $USERNAME"
            sleep 3600
```
