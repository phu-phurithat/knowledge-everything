# Secure Ingress via cert-manager + letEncrypt

This document guides you through setting up **HTTPS ingress** using:

* [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
* [cert-manager](https://cert-manager.io/)
* [Let's Encrypt](https://letsencrypt.org/)

***

### 📦 Prerequisites

* Kubernetes Cluster (e.g., GKE, EKS, K3s)
* Helm 3 installed
* A **domain name** pointing to your Ingress public IP (A/AAAA record)
* `kubectl` access to the cluster

***

### ⚙️ 1. Install NGINX Ingress Controller

```yaml
// ingress-values.yaml
---
controller:
  ingressClassResource: 
    name: external-nginx
  admissionWebhooks:
    enabled: false
  service:
    annotations:
      # change this for cluster compatiable (Assume using GKE)
      networking.gke.io/load-balancer-type: "External"
  # Required for ACME
  watchIngressWithoutClass: true
  extraArgs:
    ingress-class: external-nginx
```

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  -f external-ingress.yaml \
  --set controller.service.loadBalancerIP=<STATIC-IP>
```

> if want \<STATIC-IP> for GKE must create by running:\
> gcloud compute addresses create \<NAME> --project \<PROJECT-ID> --region \<CLUSETER-REGION>

> 🔎 Ensure the LoadBalancer IP is provisioned:

```bash
kubectl get svc -n ingress-nginx
```

### 🔐 2. Install cert-manager

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager --create-namespace \
 --set crds.enabled=true
```

***

### 📄 3. Create Let's Encrypt ClusterIssuer

You can use either **Staging** (for testing) or **Production**.

#### 🧪 Staging Issuer (safe for testing)

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: your@email.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - http01:
        ingress:
          class: external-nginx
```

#### 🚀 Production Issuer

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your@email.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: external-nginx
```

Apply with:

```bash
kubectl apply -f clusterissuer-prod.yaml
```

***

### 🌐 4. Configure Your Ingress Resource

Use staging for test before production because production it will strict rate limit

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
  namespace: default
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-staging"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - yourdomain.com
    secretName: my-app-tls
  rules:
  - host: yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app
            port:
              number: 80
```

Apply with:

```bash
kubectl apply -f ingress.yaml
```

### 🌍 5. DNS Configuration

Ensure your domain (e.g., `yourdomain.com`) has an A record pointing to your **Ingress external IP**.

To check IP:

```bash
kubectl get svc -n ingress-nginx
```

### ✅ 6. Verify Certificate

Check that the TLS certificate is:

* Issued by Let's Encrypt (staging)
* Browser can view certificate that be staging or production
* Auto-renewing (cert-manager handles this)

Also verify with:

```bash
kubectl describe certificate -n <namespace>
kubectl describe ingress <name> -n <namespace>
```

### :up: 7. Make Certificate trusted by browser

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
  namespace: default
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod" # Change this
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - yourdomain.com
    secretName: my-app-tls
  rules:
  - host: yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app
            port:
              number: 80
```

Then verify Certificate again.
