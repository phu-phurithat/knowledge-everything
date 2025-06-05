# Raft

## Raft Storage

`raft` is a **built-in consensus algorithm-based** storage backend introduced in Vault 1.4+. Must be (N-1)/2&#x20;

* Uses the **Raft protocol** to replicate Vault data across nodes.
* **Leader-based architecture** (1 active leader, others are followers).
* Supports **clustering**, **HA**, and **failover**.
* Keeps Vault **self-contained**—no need for Consul or external systems.

### 🗂️ Key Concepts

| Term                   | Description                                                         |
| ---------------------- | ------------------------------------------------------------------- |
| **Leader Node**        | Handles all writes and data coordination.                           |
| **Follower Node**      | Reads from the leader and replicates its data.                      |
| **Quorum**             | Minimum number of nodes needed for consensus (typically `N/2 + 1`). |
| **Peers**              | Nodes participating in the Raft cluster.                            |
| **Integrated Storage** | Vault’s internal use of Raft to persist and replicate secrets data. |

### 🛠️ Sample `vault.hcl` Config (Raft First node -> Leader)

<pre class="language-hcl"><code class="lang-hcl">listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = 1  # Use real TLS certs in production
}

storage "raft" {
  path    = "/opt/vault/data"
<strong>  node_id = "vault-node-1" # This must unique
</strong>}

<strong>api_addr     = "http://&#x3C;this-node-ip>:8200"
</strong><strong>cluster_addr = "http://&#x3C;this-node-ip>:8201"
</strong></code></pre>

### Clustering with Raft

To add a second node(Followers):

1.  Start the new Vault server with its own `vault.hcl`:

    ```hcl
    listener "tcp" {
      address     = "0.0.0.0:8200"
      tls_disable = 1  # Use real TLS certs in production
    }

    storage "raft" {
      path    = "/opt/vault/data"
      node_id = "vault-node-2" # This must unique
    }

    api_addr     = "http://<this-node-ip>:8200"
    cluster_addr = "http://<this-node-ip>:8201"
    ```
2.  On the new node:

    ```bash
    vault operator raft join http://<leader-node-ip>:8200
    ```
3. Unseal the new node with the same unseal keys.

### 📘 Additional Tips

* You must unseal **each node**.
* The leader handles all writes; followers replicate.
*   Monitor cluster health with:

    ```bash
    vault operator raft list-peers
    ```

### 🔄 Vault Raft Cluster Commands

| Command                                       | Description                          |
| --------------------------------------------- | ------------------------------------ |
| `vault operator raft list-peers`              | List current cluster members         |
| `vault operator raft remove-peer node_id`     | Remove a node from the cluster       |
| `vault operator raft autopilot get-config`    | View AutoPilot (self-healing) config |
| `vault operator raft snapshot save snap.snap` | Backup current Raft state            |

### 📌 Quorum and Availability

For a healthy HA setup:

* Use **3 or 5 Vault nodes** (odd number preferred).
* **Quorum** is required for:
  * Unseal operations
  * Writes
  * Configuration changes

> ℹ️ In a 3-node cluster, **at least 2 nodes must be online** for Vault to function fully.

### 📎 Best Practices

| Practice                                              | Reason                             |
| ----------------------------------------------------- | ---------------------------------- |
| Use an odd number of nodes                            | Ensures quorum is achievable       |
| Use static IPs or DNS names                           | Stability in cluster communication |
| Regularly take Raft snapshots                         | Reliable backups                   |
| Automate unsealing (e.g., with AWS KMS, GCP KMS, HSM) | Avoid manual intervention          |
| Monitor leader status                                 | Detect failovers early             |
