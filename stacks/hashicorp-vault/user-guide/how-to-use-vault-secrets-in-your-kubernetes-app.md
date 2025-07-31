# How to Use Vault Secrets in Your Kubernetes App

### What is Vault?

HashiCorp Vault securely stores and manages secrets (passwords, tokens, keys). Your Kubernetes app can fetch secrets from Vault **without hardcoding them**.

***

### Step 1: Set Up Your Kubernetes ServiceAccount

Your app pods must use a Kubernetes ServiceAccount trusted by Vault.

Example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  namespace: default
```

Apply it:

```bash
kubectl apply -f serviceaccount.yaml
```

***

### Step 2: Deploy Your App with Vault Agent Sidecar Injection

Vault Agent runs alongside your app in the pod and automatically fetches secrets from Vault.

#### Add these annotations to your Pod/Deployment metadata:

```yaml
metadata:
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "my-app-role"
    vault.hashicorp.com/agent-inject-secret-config.txt: "kv/data/my-app/config"
    vault.hashicorp.com/agent-inject-template-config.txt: |
      {{- with secret "kv/data/my-app/config" -}}
      export USERNAME={{ .Data.data.username }}
      export PASSWORD={{ .Data.data.password }}
      {{- end }}
```

Your container spec must use the ServiceAccount:

```yaml
spec:
  serviceAccountName: my-app-sa
```

***

### Step 3: Access Secrets in Your Application

Vault Agent writes secrets to `/vault/secrets/config.txt` in the pod.

In your app’s code or startup script, read the secrets like:

```bash
source /vault/secrets/config.txt
echo "Database user is $USERNAME"
```

***

### Step 4: Vault Admins’ Responsibilities (For Your Info)

* Create a Vault policy (e.g., `my-app-policy`) granting read access to your secrets path.
* Create a Kubernetes Auth role binding your ServiceAccount (`my-app-sa`) to that policy.
* Store your app secrets in Vault at path `kv/data/my-app/config`.
* Configure Vault Kubernetes Auth method with the Kubernetes cluster info.

***

### Step 5: What Happens Automatically?

* Vault Agent authenticates to Vault using your pod’s ServiceAccount token.
* Vault issues a short-lived Vault token.
* Vault Agent fetches secrets and writes them to files.
* Vault Agent auto-renews tokens and secrets without your app needing to do anything.

***

### Troubleshooting

* Check Vault Agent logs:

```bash
bashCopyEditkubectl logs <pod-name> -c vault-agent
```

* Verify the ServiceAccount name matches the Vault role.
* Confirm your Vault policy allows reading the secret.
* Ensure Vault server is reachable.

***

### Summary Checklist for App Developers

| Task                                   | Done? |
| -------------------------------------- | ----- |
| Use the correct ServiceAccount         | ⬜     |
| Add Vault annotations to pod           | ⬜     |
| Read secrets from `/vault/secrets`     | ⬜     |
| Don’t hardcode secrets or Vault tokens | ⬜     |
