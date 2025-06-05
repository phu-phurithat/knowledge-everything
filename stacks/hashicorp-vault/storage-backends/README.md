# Storage Backends

Vault uses a **storage backend** to persist all its data, including:

* Secrets
* Policies
* Tokens
* Leases
* Audit logs (if configured)

### Storage Backends Overview

Vault supports multiple storage backends for persisting data. Common choices:

| Backend    | Use Case                           | Notes                                  |
| ---------- | ---------------------------------- | -------------------------------------- |
| `file`     | Simple local testing               | Not HA                                 |
| `raft`     | Built-in, recommended for HA       | Production-ready, supports clustering  |
| `consul`   | Advanced HA with Consul as backend | Complex, but reliable and scalable     |
| `s3`       | External storage (read-only)       | Best used as seal storage, not backend |
| `dynamodb` | AWS-native option                  | Needs IAM setup                        |

{% hint style="info" %}
Note: **Vault does not use external databases.** Instead, it supports various pluggable backends like Raft, Consul, etc.
{% endhint %}

### File Storage (Simple)

```hcl
storage "file" {
  path = "/opt/vault/data"
}
```
