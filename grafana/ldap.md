# LDAP

## Grafana LDAP Setup with Helm Values Only

***

### ✅ Goal

Enable LDAP login in Grafana using **Helm chart values only**, leveraging:

* `ldap.enabled: true`
* `ldap.existingSecret: <your-secret>`
* Secret contains `ldap.toml` config

***

### 🪪 1. Prepare `ldap.toml`

Create your `ldap.toml` file locally with the LDAP connection details: (This is free LDAP server)

```toml
# To troubleshoot and get more log info enable ldap debug logging in grafana.ini
# [log]
# filters = ldap:debug

[[servers]]
# Ldap server host (specify multiple hosts space separated)
host = "ldap.forumsys.com"
# Default port is 389 or 636 if use_ssl = true
port = 389
# Set to true if ldap server supports TLS
use_ssl = false
# Set to true if connect ldap server with STARTTLS pattern (create connection in insecure, then upgrade to secure connection with TLS)
start_tls = false
# set to true if you want to skip ssl cert validation
ssl_skip_verify = false
# set to the path to your root CA certificate or leave unset to use system defaults
# root_ca_cert = "/path/to/certificate.crt"
# Authentication against LDAP servers requiring client certificates
# client_cert = "/path/to/client.crt"
# client_key = "/path/to/client.key"

# Search user bind dn
bind_dn = "cn=read-only-admin,dc=example,dc=com"
# Search user bind password
# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
bind_password = 'password'

# User search filter, for example "(cn=%s)" or "(sAMAccountName=%s)" or "(uid=%s)"
search_filter = "(uid=%s)"

# An array of base dns to search through
search_base_dns = ["dc=example,dc=com"]

## For Posix or LDAP setups that does not support member_of attribute you can define the below settings
## Please check grafana LDAP docs for examples
group_search_filter = "(&(objectClass=groupOfUniqueNames)(uniqueMember=uid=%s,dc=example,dc=com))"
group_search_base_dns = ["dc=example,dc=com"]
group_search_filter_user_attribute = "uid"

# Specify names of the ldap attributes your ldap uses
[servers.attributes]
name = "givenName"
surname = "sn"
username = "uid"
member_of = "DN"
email =  "mail"

# Map ldap groups to grafana org roles
[[servers.group_mappings]]
group_dn = "ou=chemists,dc=example,dc=com"
org_role = "Admin"
# To make user an instance admin  (Grafana Admin) uncomment line below
# grafana_admin = true
# The Grafana organization database id, optional, if left out the default org (id 1) will be used
# org_id = 1

[[servers.group_mappings]]
group_dn = "ou=scientists,dc=example,dc=com"
org_role = "Editor"

[[servers.group_mappings]]
# If you want to match all (or no ldap groups) then you can use wildcard
group_dn = "*"
org_role = "Viewer"
```

***

### 🔐 2. Create Kubernetes Secret with `ldap.toml`

```bash
kubectl create secret generic grafana-ldap-toml \
  --from-file=ldap.toml=./ldap.toml \
  -n monitoring
```

> Replace `monitoring` with your Grafana namespace if needed.

***

### ⚙️ 3. Create `values.yaml` for Helm

Here’s the **minimum config** to enable LDAP using that secret:

```yaml
ldap:
  config: ''
  enabled: true
  existingSecret: grafana-ldap-toml
```

***

### 🚀 4. Deploy Grafana with Helm

If not installed yet:

```bash
helm install grafana grafana/grafana -f values.yaml -n monitoring
```

If already installed:

```bash
helm upgrade grafana grafana/grafana -f values.yaml -n monitoring
```

***

### 🧪 5. Test LDAP Login

* Open Grafana UI
* Try logging in with your LDAP user credentials
* You should see users automatically created in Grafana with appropriate roles.
