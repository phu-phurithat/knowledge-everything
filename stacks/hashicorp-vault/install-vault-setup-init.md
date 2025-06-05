# Install Vault, setup, init

## Install guildline&#x20;

in [Official HashiCorp Doc](https://developer.hashicorp.com/vault/install)&#x20;

## Setup: Running Vault

### Dev Mode (for local dev/testing)

```bash
vault server -dev
```

> Auto-unsealed with root token printed in console. Don't use in production.

### Production Mode (Manual + Config File)

To start with production mode must config in `vault.hcl`

**Example: `vault.hcl`**

```hcl
listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = 1  # For dev only; enable TLS in prod
}

storage "file" {
  path = "/opt/vault/data"
}

api_addr = "http://<your-ip>:8200"
```

And config env: `VAULT_ADDR`

```bash
export VAULT_ADDR=http://<your-ip>:8200
```

Start with: (Testing config)

```bash
vault server -config=/etc/vault.d/vault.hcl
```

> My opinion in Linux should start vault by:&#x20;

```bash
sudo systemctl start vault # start vault by systemctl
sudo systemctl enable vault # make vault start on boot
```

### Initialize Vault&#x20;

```bash
vault operator init
```

Default initial vault give 5 unseal keys, 1 root token. To unseal must contain 3 in 5 keys.

Unseal:

```bash
vault operator unseal <key>
```

Login:&#x20;

```bash
vault login <initial_root_token>
```
