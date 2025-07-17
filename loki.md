# Loki

### 📘 Loki Production Deployment Documentation (Updated 17/7/2025)

#### 1. 📦 Architecture Overview

**Components:**

* **Loki** – log aggregation backend
* **Promtail** – agent that ships logs to Loki
* **Grafana** – UI to query and visualize logs
* **Kong Gateway** – exposes Loki via HTTPS and provides auth
* **Storage** – S3(Minio)

**Data Flow:**

```
[Application Pods] → Promtail → Loki → Grafana → via Kong HTTPS
```

***

#### 2. 🚀 Deployment Details

**⚙️ Loki Helm Chart**

* **Chart**: `grafana/loki`
* **Version**: 3.5.0
* **Namespace**: `monitoring`
* **Storage**: S3
* **Configuration**:
  * Enabled multi-tenancy: `auth_enabled: true`
  * TLS: Handled by Kong at ingress
  * Authentication: Delegated to Kong (e.g., basic-auth or LDAP)

```yaml
# loki-values.yaml
chunksCache:
  allocatedMemory: 1024

loki:
  auth_enabled: true
  
  limits_config:
    max_series_per_query: 2000  # or higher if needed

  ring:
    kvstore:
      store: memberlist

  schemaConfig:
    configs:
      - from: "2024-04-01"
        store: tsdb
        object_store: s3
        schema: v13
        index:
          prefix: loki_index_
          period: 24h

  storage_config:
    tsdb_shipper:
      active_index_directory: /var/loki/index
      cache_location: /var/loki/cache
    aws:
      bucketnames: loki-logs-prod
      endpoint: minio-hl.minio.svc.cluster.local:9000
      region: us-east-1
      access_key_id: minioadmin
      secret_access_key: miniostorage
      insecure: false
      http_config:
        idle_conn_timeout: 90s
        response_header_timeout: 0s
        insecure_skip_verify: true
      s3forcepathstyle: true

  ruler:
    enable_api: true 

  ingester:
    chunk_encoding: snappy

  pattern_ingester:
    enabled: true

  limits_config:
    allow_structured_metadata: true
    volume_enabled: true
    retention_period: 168h # 7 days

  compactor:
    retention_enabled: true
    delete_request_store: s3

  querier:
    max_concurrent: 16

  storage:
    type: s3
    bucketNames: 
      chunks: loki-logs-prod
      ruler: loki-ruler-prod
    s3:
      endpoint: minio-hl.minio.svc.cluster.local:9000
      region: us-east-1
      accessKeyId: minioadmin
      secretAccessKey: miniostorage
      s3ForcePathStyle: true
      insecure: false
      http_config:
        idle_conn_timeout: 90s
        response_header_timeout: 0s
        insecure_skip_verify: true

serviceAccount:
  create: true
  annotations: {}

deploymentMode: Distributed

ingester:
  replicas: 3 # Shloud be 2n+1
  zoneAwareReplication:
    enabled: false

querier:
  replicas: 2
  maxUnavailable: 1

queryFrontend:
  replicas: 2
  maxUnavailable: 1

queryScheduler:
  replicas: 2
  maxUnavailable: 1

distributor:
  replicas: 2
  maxUnavailable: 1

compactor:
  replicas: 1

indexGateway:
  replicas: 2
  maxUnavailable: 1

ruler:
  replicas: 2
  maxUnavailable: 1

gateway:
  service:
    type: ClusterIP

# Disable built-in minio
minio:
  enabled: false

backend:
  replicas: 0
read:
  replicas: 0
write:
  replicas: 0

singleBinary:
  replicas: 0
```

**🛠 Promtail**

* **Deployed as**: DaemonSet
* **Namespace**: `monitoring`
* **Target Logs**: `/var/log/pods`, `/var/log/containers`
*   **Clients URL**: Pointing to Kong:

    ```yaml
    clients:
      - url: https://<kong-url>/loki/api/v1/push
    ```
* Values:

```yaml
# promtail-values.yaml
image:
  repository: grafana/promtail
  tag: "latest"  # or your desired version

config:
  clients:
    - tenant_id: "1" 
      url: http://loki-gateway/loki/api/v1/push
  scrape_configs:
    - job_name: kubernetes-pods
      kubernetes_sd_configs:
        - role: pod
      pipeline_stages:
        - docker: {}
      relabel_configs:
        - source_labels: [__meta_kubernetes_pod_label_app]
          action: keep
          regex: .+
        - source_labels: [__meta_kubernetes_pod_container_name]
          target_label: container
        - source_labels: [__meta_kubernetes_namespace]
          target_label: namespace
        - source_labels: [__meta_kubernetes_pod_name]
          target_label: pod

```

***

#### 3. 🌐 Kong Gateway (Assume)

* **Exposes**: `/loki/` path to internal `loki.monitoring.svc.cluster.local:3100`
* **TLS**: Enabled via cert-manager or self-signed certs
* **Auth Plugins**:
  * Basic Auth (or LDAP)
*   **Ingress** Example:

    ```yaml
    yamlCopyEditapiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: loki-ingress
      annotations:
        konghq.com/plugins: basic-auth
    spec:
      rules:
        - host: logs.example.com
          http:
            paths:
              - path: /loki
                pathType: Prefix
                backend:
                  service:
                    name: loki
                    port:
                      number: 3100
    ```

***

#### 4. 🔐 Authentication

* Kong handles auth
  * **LDAP plugin** or **Basic Auth** depending on needs
* Loki itself runs with `auth_enabled: true` to support tenants (if needed)

***

#### 5. 🧪 Ruler & Alerting (Optional)

* Loki Ruler enabled for alerting
* Alertmanager can be integrated via Grafana Mimir or Prometheus

***

#### 6. 🔍 Querying Logs

In Grafana:

*   Add Loki data source with:

    ```
    https://logs.example.com/loki
    ```
* Use `X-Scope-OrgID` header for multi-tenancy enabled

***

#### 7. 🛡️ Production Hardening

* Rate limiting at Kong
* TLS enabled
* Minimal Promtail permissions (read logs only)
* Use persistent storage (or cloud object store)

***

#### 8. 🔧 Maintenance Tips

* Rotate logs using retention configs
* Monitor Loki and Promtail health (`/metrics`)
* Watch disk space and CPU usage on ingest nodes
