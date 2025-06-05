# limits and maximums

## **General Concept**

* Vault has various types of limits:
  * **Fixed limits** (hardcoded in Vault).
  * **Configurable limits** (can be changed via settings).
  * **Backend-specific limits** (based on Consul or Raft).
* **Performance degradation may occur** before reaching the absolute limits.

***

## Storage-Related limits <a href="#storage-related-limits" id="storage-related-limits"></a>

### 💾 2. Storage Entry Size

In Vault, **everything is stored as entries in a storage backend** — like secrets, policies, tokens, etc. Each of these is called a **storage entry**.

The **storage entry size** is the **maximum size** (in bytes) of a single object that Vault can write to its storage backend.

* **Raft (Integrated Storage)**:
  * Default max entry size: **1 MiB**
  * Entries >512 KiB are **automatically chunked**.
  * You can increase with: `max_entry_size`
* **Consul**:
  * Default max entry size: **512 KiB**
  * **No chunking** – all data written as one large object.
  * Increased via: `kv_max_value_size` in Consul config.

⚠️ **Warning**:

* Large entries in Consul = **high read-modify-write cost**, performance issues, or cluster instability.
* If entry size is too large → **Vault errors**, needs config adjustment.

***

### 📦 **3. Mount Point Limits**

#### 🧩 What is a Mount Point?

A **mount point** is where a secret engine or auth method is enabled (e.g., `secret/`, `auth/github/`).

#### 🔸 Limits

* Each mount (auth or secret) must **fit into a single storage entry**.
* Rough space usage:
  * Raw: \~500 bytes per mount
  * Compressed: \~75 bytes per mount

| Storage Backend        | Max Mount Points |
| ---------------------- | ---------------- |
| **Consul (512 KiB)**   | \~7,000          |
| **Integrated (1 MiB)** | \~14,000         |

* No enforced limit on mount point **name length**, but **longer paths** increase size usage.
* Tracked via:
  * `sys/auth`
  * `sys/mounts`
  * Telemetry: `vault.core.mount_table.num_entries` and `vault.core.mount_table.size`

***

### 🧱 **4. Namespace Limits**

#### 🔸 Concepts

* Namespaces = used for **multitenancy** in Vault.
* Each namespace must have:
  * `sys` and `identity` mounts
  * One **local** secret engine (`cubbyhole`)
  * One **auth method** (`token`)

#### 🔸 Limits

| Storage Backend | Max Namespaces | With Extra Secret Engine |
| --------------- | -------------- | ------------------------ |
| **Consul**      | \~3,500        | \~2,300                  |
| **Integrated**  | \~7,000        | \~4,600                  |

* **Namespace depth (nesting)**:
  * Consul: \~160
  * Integrated: \~220
  * Calculated at \~40 bytes per level of depth.

Tracked via:

* `sys/namespaces`
* Estimate: Divide mount point limit by number of mounts per namespace

***

### 👥 **5. Identity Entities & Groups Limits**

#### 🧍 Identity Entities

* Metadata constraints:
  * **Max 64** key-value pairs
  * **Key size**: 128 bytes
  * **Value size**: 512 bytes
* Vault shards identities into **256 storage entries**.

> So, total storage space:
>
> * **Consul**: 128 MiB
> * **Integrated**: 256 MiB

* **Entity aliases** are stored within the entity object.

#### 🔢 Entity Capacity Estimates

| Case                           | Consul (512 KiB) | Integrated (1 MiB) |
| ------------------------------ | ---------------- | ------------------ |
| Best case (200B/entity)        | \~610,000        | \~1,250,000        |
| Conservative (500B/entity)     | \~250,000        | \~480,000          |
| Max metadata (\~41 KiB/entity) | \~670            | \~2,400            |

Tracked via telemetry:

* `vault.identity.num_entities`
* `vault.identity.entities.count`

***

#### 👥 Groups

* Groups also use 256 storage entries (separate from entities).
* Aliases & members are stored inline.
* Storage usage example:
  * 10 members, no metadata → \~500B
  * 100 members → \~4,000B

| Case                     | Consul    | Integrated |
| ------------------------ | --------- | ---------- |
| Groups with 10 members   | \~250,000 | \~480,000  |
| Groups with 100 members  | \~22,000  | \~50,000   |
| Max members in one group | \~11,500  | \~23,000   |

⚠️ **Very large groups (>1000 members)** should be avoided — the entire member list must fit in one entry. Use **external groups or subgroups**.

***

### 🔐 **6. Token Limits**

* Each token = **one storage entry**
* No hard limit on number of tokens
* Metadata size constraints:
  * **No limit** on number of metadata key-value pairs
  * **No limit** on key or value size
  * **Total size** of token metadata must be **≤ 512 KiB**

***

### 📜 **7. Policy Limits**

Policies are stored in Vault storage, so their size is limited by the backend:

| Type                                | Consul (512 KiB) | Integrated (1 MiB) |
| ----------------------------------- | ---------------- | ------------------ |
| Max policy size                     | 512 KiB          | 1 MiB              |
| Max policies per namespace          | No limit         | No limit           |
| Max policies per token/entity/group | \~14,000         | \~28,000           |

⚠️ **Performance Warning**: When a token is used, Vault assembles all policies from:

* The token
* The associated entity
* Groups that entity belongs to
* Nested groups

Large numbers of policies may slow response time.

**Monitor** using: `vault.core.fetch_acl_and_token`

***

### 🗝️ **8. Versioned Key-Value Store (kv-v2 Secret Engine)**

* Number of secrets: **Unlimited**, limited only by storage capacity.
* Max size per version of a secret:
  * Slightly less than one storage entry:
    * **Consul:** 512 KiB
    * **Integrated:** 1 MiB
* Number of versions per secret:
  * Default: **10**
  * Configurable (can be more, tested at least 24,000 versions)
* Each version must fit in a **single storage entry** (data stored as JSON).
* Version metadata consumes **21 bytes per version** and must fit in a separate storage entry.
* Secrets also have version-agnostic metadata, including custom metadata:

| Limit                                     | Value     |
| ----------------------------------------- | --------- |
| Number of custom metadata key-value pairs | 64        |
| Custom metadata key size                  | 128 bytes |
| Custom metadata value size                | 512 bytes |

***

### 🔐 **9. Transit Secret Engine Limits**

* Maximum ciphertext or plaintext size limited by Vault’s **max\_request\_size** (default 32 MiB).
* All archived versions of a single key must fit in a **single storage entry**.
* Key size limits depend on algorithm and backend:

| Key Type        | Consul (512 KiB) | Integrated (1 MiB) |
| --------------- | ---------------- | ------------------ |
| aes128-gcm96    | 2008 bytes       | 4017 bytes         |
| aes256-gcm96    | 1865             | 3731               |
| chacha-poly1305 | 1865             | 3731               |
| ed25519         | 1420             | 2841               |
| ecdsa-p256      | 817              | 1635               |
| ecdsa-p384      | 659              | 1318               |
| ecdsa-p523      | 539              | 1078               |
| 1024-bit RSA    | 169              | 333                |
| 2048-bit RSA    | 116              | 233                |
| 4096-bit RSA    | 89               | 178                |

***

## Other limits <a href="#other-limits" id="other-limits"></a>

### 🚀 **10. Request Limits**

* **Max HTTP request size:** Controlled by `max_request_size` in the listener stanza.
  * Default: **32 MiB**
  * Limits the maximum size of:
    * Transit operations
    * Key-value secrets
* **Max request duration:** Controlled by `max_request_duration`
  * Default: **90 seconds**
  * If an operation takes longer, Vault returns failure.
* Client-side timeout via environment variable `VAULT_CLIENT_TIMEOUT`
  * Default: **60 seconds**

***

### 🏢 **11. Cluster and Replication Limits**

* No hard limits on:
  * Maximum cluster size
  * Number of DR replicas
  * Number of performance replicas
* But, each replica adds overhead to the active node because writes must be duplicated.
* High number of replicas may cause:
  * CPU and network strain
  * Lag between write-ahead logs (WAL) on primary and replicas
* Monitor:
  * Active node’s CPU and network usage
  * WAL lag

***

### ⏳ **12. Lease Limits**

#### 📦 What is a Lease?

A **lease** is a metadata record attached to certain secrets (or tokens) that tells Vault:

* **How long the data is valid**
* Whether it can be **renewed**
* When it should be **revoked**

#### **Limits**

* Max number of leases (tokens, secrets, etc.) — advisory limit around **256,000**
* Max lease or token duration — default **768 hours** (32 days)
* High lease counts can degrade performance.
* Recommended: use short TTLs to avoid backlogs.
* Monitor unexpired leases with `vault.expire.num_leases` metric.

***

### 🔄 **13. Transform Limits**

* The Transform secret engine follows **FF3-1** standards for minimum and maximum input lengths.
* Limits depend on the size of the alphabet used for transformations.

***

### 🔌 **14. External Plugin Limits**

* Vault launches a **separate process** per external plugin.
* Each enabled secret engine or auth method that uses plugins runs its own process.
* Database secrets engines spawn a process per configured database connection.
* Each plugin process uses CPU, memory, networking, and file descriptors.
* No fixed limit on the number of plugins or connections.
* Resource consumption grows roughly **linearly** with number of plugin processes.
* Practical limits depend on system resources and usage patterns.
