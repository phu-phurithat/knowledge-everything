---
description: >-
  Vault have used in Secrets Management. HashiCorp Vault is a tool for secrets
  management, encryption, and access control in modern infrastructure. It solves
  key challenges in securely storing, accessin
---

# HashiCorp Vault

## Core Use Cases

1. #### **Secrets Management**
   * Store API keys, passwords, tokens, database credentials securely.
   * Access secrets programmatically via CLI, HTTP API, or SDKs.
   * Support for versioning, TTLs, and dynamic secrets.
2. #### **Encryption as a Service**
   * Encrypt and decrypt data using the **Transit secrets engine**.
   * Offload crypto operations from your application.
   * Useful for database field encryption, application-level security, etc.
3. #### **Dynamic Secrets**
   * Vault can generate secrets (e.g., DB credentials, AWS IAM keys) on demand.
   * Secrets expire automatically—no need to rotate manually.
   * Example: Create a short-lived MySQL user with read-only access.
4. #### **Identity-Based Access**
   * Supports multiple auth methods: Kubernetes, GitHub, LDAP, AppRole, etc.
   * Access is managed via fine-grained **policies**.
   * Only allow apps/users to access what they need.
5. #### **Data Protection & Key Management**
   * Acts as a **Key Management System (KMS)**.
   * Integrates with AWS KMS, GCP KMS, HSMs.
   * Supports use cases like envelope encryption, TDE master key storage.
6. #### **Multi-Cloud and Zero Trust Security**
   * Centralized secrets across AWS, GCP, Azure, on-prem.
   * Enforces least privilege and identity-based access control across environments.

| Use Case                      | Example Scenario                                |
| ----------------------------- | ----------------------------------------------- |
| Store & access app secrets    | An app reads its DB password from Vault KV      |
| Encrypt sensitive fields      | Encrypt SSNs or credit card fields with Transit |
| Short-lived cloud credentials | Auto-generate AWS IAM credentials with TTL      |
| CI/CD secrets injection       | GitHub Actions pulls secrets via Vault AppRole  |
| Secrets in Kubernetes         | Vault Agent injects secrets into pods           |
| MariaDB TDE                   | Vault acts as a key source for DB encryption    |
