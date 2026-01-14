---
title: Building an Internal CA for your Homelab with HashiCorp Vault
date: 2026-01-13 18:00:00 +0100
description: "Deploy Vault as your own CA using Portainer - say goodbye to self-signed certificate warnings"
tags:
- devops
- homelab
- security
- vault
showComments: true
---

Welcome to another episode of my homelab documentation series. If you’re running multiple services in your homelab, like I do, you’ve probably wrestled with self‑signed certificates, browser warnings, and the hassle of renewing TLS certs. [HashiCorp Vault’s](https://www.hashicorp.com/en/products/vault) PKI secrets engine can solve all of that, acting as your own internal Certificate Authority (CA) with automated certificate issuance and renewal, it is a powerful tool for managing secrets, encryption keys, and identity-based access to sensitive data. Vault came some time ago as recommendation from a good friend of mine, he was already running it as CA on his homelab so I decided to give it a try.

In this guide, I'll walk through deploying Vault in your homelab using **Portainer** with **TLS enabled**, so you can access it securely over HTTPS. This guide is intentionally detailed so you can follow along exactly the same procedure I used to deploy Vault in my homelab.

## Why Run Vault in Your Homelab?

- **Learn production practices**: experiment with TLS, unsealing, and persistent storage.
- **Integrate with other services**: test Vault as a backend for Kubernetes, Docker, or personal projects.
- **Build confidence**: understand operational workflows before deploying to production.

## Prerequisites

- A server or VM running Docker + Portainer.
- Basic understanding of Docker volumes.
- TLS certificates.

For out initial setup, we’ll generate self-signed certificates and place everything under `/opt/vault` in our Docker host, we will replace this self-signed certificates with final Vault certificates later. First create the directories in the Docker host.

```text
sudo mkdir -p /opt/vault/{config,certs,data}
sudo chown $(id -u):$(id -g) /opt/vault -R
cd /opt/vault/certs
```

This creates:

- `/opt/vault/config` → configuration files.
- `/opt/vault/certs` → TLS certs.
- `/opt/vault/data` → persistent storage.

Create a small `openssl` config.

```ini
cat > /opt/vault/certs/vault-openssl.cnf <<'EOF'
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
CN = vault.starlabs.local

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = vault.starlabs.local
DNS.2 = localhost
IP.1 = 127.0.0.1
IP.2 = 192.168.1.20
EOF
```

Add your homelab IP as `IP.2` value under `[alt_names]` before creating the cert, e.g. I added `IP.2 = 192.168.1.50` since that's the IP address of my Portainer instance.

Generate the key, CSR and self-signed certs:

```text
cd /opt/vault/certs
openssl req -new -nodes -newkey rsa:2048 \
  -keyout vault.key -out vault.csr -config vault-openssl.cnf

openssl x509 -req -in vault.csr -signkey vault.key \
  -out vault.crt -days 3650 -extensions v3_req -extfile vault-openssl.cnf

rm vault.csr
chmod 600 vault.key
```

## Vault configuration

Create `/opt/vault/config/config.hcl`:

```hcl
ui = true
api_addr     = "https://vault.starlabs.local:8200"
cluster_addr = "https://vault.starlabs.local:8201"

listener "tcp" {
  address       = "0.0.0.0:8200"
  tls_cert_file = "/vault/certs/vault.crt"
  tls_key_file  = "/vault/certs/vault.key"
}

storage "file" {
  path = "/vault/data"
}
```

- `ui=true` - Enables Vault web UI.
- `listener "tcp"` - Listens on all interfaces, port 8200, with TLS enabled.
- `storage "file"` - tores Vault data in the mounted volume (`/opt/vault/data`).

## Portainer Deployment

1. Open **Portainer**.
2. Navigate to **Stacks -> Add Stack**.
3. Name the stack `vault`.
4. Paste the below YAML. You can adjust host paths if you used different directories.

    ```yml
    version: '3.9'
    services:
      vault:
        image: hashicorp/vault:latest
        container_name: vault
        restart: unless-stopped
        cap_add:
          - IPC_LOCK
        ports:
          - "8200:8200"
        environment:
          VAULT_ADDR: https://vault.starlabs.local:8200
        volumes:
          - /opt/vault/config:/vault/config:ro
          - /opt/vault/certs:/vault/certs:ro
          - /opt/vault/data:/vault/data
          - /dev/null:/etc/vault.d   # mask defaults
        entrypoint: ["vault"]         # ensure only vault runs
        command: ["server","-config=/vault/config/config.hcl"]
    ```

5. Deploy the stack.

In Portainer watch logs and ensure the container enters running state. Vault should now be running as a container with TLS enabled.

### Portainer Troubleshooting Checklist for Vault

When deploying Vault in Portainer, you may encounter issues that don’t happen when running `docker run` manually, actually I run into some of them during my Vault setup. Here are common pitfalls and fixes:

#### Double Vault Processes → “address already in use”

- **Symptom:** Logs show

  ```text
  Error initializing listener of type tcp: listen tcp4 0.0.0.0:8200: bind: address already in use
  ```

- **Cause:** Portainer/Compose runs the default Vault `docker-entrypoint.sh`, which launches Vault with configs from `/etc/vault.d`. Your stack command then starts a second Vault process → port conflict.
- **Fix:** Override the entrypoint and mask default configs:

  ```yaml
  entrypoint: ["vault"]
  command: ["server", "-config=/vault/config/config.hcl"]
  volumes:
    - /dev/null:/etc/vault.d   # masks default configs
  ```

#### TLS Key Permission Denied

- **Symptom:**

  ```text
  error loading TLS cert: open /vault/certs/vault.key: permission denied
  ```

- **Cause:** Container’s `vault` user can’t read the private key.
- **Fix:**

  ```bash
  sudo chown root:vault /opt/vault/certs/vault.key
  sudo chmod 640 /opt/vault/certs/vault.key
  ```

#### `mlock` / IPC_LOCK Errors

- **Symptom:**

  ```text
  Failed to lock memory: cannot allocate memory
  ```

- **Cause:** `mlock` isn’t available inside most Docker environments.
- **Fix:** Add capability and/or disable mlock in config:

  ```yaml
  cap_add:
    - IPC_LOCK
  ```

  And in `config.hcl`:

  ```hcl
  disable_mlock = true
  ```

#### Check Config Before Deploy

- Run Vault in debug/verify mode outside Portainer before stack deployment:

  ```bash
  docker run --rm -it \
    --cap-add=IPC_LOCK \
    -v /opt/vault/config:/vault/config:ro \
    -v /opt/vault/certs:/vault/certs:ro \
    -v /opt/vault/data:/vault/data \
    hashicorp/vault server -config=/vault/config/config.hcl
  ```

## Initialize and Unseal Vault

Once the container is running we need to initialize Vault.

Check status by running this inside the container:

```text
docker exec -it vault vault status
```

You may see a TLS error like this:

```text
Error checking seal status: Get "https://vault.starlabs.local:8200/v1/sys/seal-status": 
tls: failed to verify certificate: x509: certificate signed by unknown authority
```

This is expected when using a self-signed or custom CA. You can bypass it for bootstrap/testing with:

```text
docker exec -it vault vault status -tls-skip-verify
```

Example output:

```text
Key                Value
---                -----
Seal Type          shamir
Initialized        false
Sealed             true
Total Shares       0
Threshold          0
Unseal Progress    0/0
Unseal Nonce       n/a
Version            1.20.3
Build Date         2025-08-27T10:53:27Z
Storage Type       file
HA Enabled         false
```

If Vault is uninitialized, initialize and save the keys (example uses 1 share/threshold for simplicity — for real setups use multiple shares):

```text
docker exec -i vault vault operator init -key-shares=1 -key-threshold=1 -format=json -tls-skip-verify > /opt/vault/init.json
```

Store the `init.json` somewhere safe (it contains unseal key(s) and root token). Extract values:

```text
jq -r '.unseal_keys_b64[]' /opt/vault/init.json > /opt/vault/unseal.key
jq -r '.root_token' /opt/vault/init.json > /opt/vault/root.token
chmod 600 /opt/vault/unseal.key /opt/vault/root.token
```

Unseal:

```bash
for key in `cat /opt/vault/unseal.key`; do 
  docker exec -i vault vault operator unseal -tls-skip-verify $key
done
```

Login with the root token (inside the container):

```bash
ROOT=$(cat /opt/vault/root.token)
docker exec -i vault vault login -tls-skip-verify "$ROOT"
```

I've created a small script in `bash` to unseal Vault after each restart, replace the hostname and paths with your own values.

```bash
#!/user/bin/env bash

# Path to your init.json file
UNSEAL_FILE="/opt/vault/unseal.key"
UNSEAL_KEYS=$(cat $UNSEAL_FILE)

# Vault address (should match your config)
export VAULT_ADDR="https://vault.starlabs.local:8200"

# If using self-signed certs, you may need this
export VAULT_SKIP_VERIFY=true

echo "Starting Vault unseal process..."

# Loop through each key and unseal
for key in $UNSEAL_KEYS; do
  echo "Applying unseal key..."
  docker exec -i vault vault operator unseal -tls-skip-verify $key
done

echo "Vault should now be unsealed."
```

## Verify TLS from the host

If you used a self-signed cert, point curl to the cert:

```text
curl --cacert /opt/vault/certs/vault.crt https://vault.starlabs.local:8200/v1/sys/health
```

You can also set:

```bash
export VAULT_ADDR='https://vault.starlabs.local:8200'
export VAULT_CACERT='/opt/vault/certs/vault.crt'
vault status    # run locally if Vault CLI is installed
```

## Access Vault UI

Open `https://vault.starlabs.local:8200`

Your browser may warn about the self-signed cert. Import `vault.crt` into your OS/browser trust store to avoid warnings, but is not mandatory since we will generate a certificate for Vault later.

## Enable PKI and Configure CA

Since we are still using a self-signed cert, we still need to use the `-tls-skip-verify` flag or `VAULT_SKIP_VERIFY=true` env var inside the container.

### Bootstrap PKI

First execute a shell inside the container:

```bash
docker exec -it vault sh
```

Once you are inside the container, enable the PKI secrets engine and configure it:

```bash
export VAULT_SKIP_VERIFY=true
vault secrets enable pki
```

### Configure Vault as Root CA

Create the root CA. Still with `VAULT_SKIP_VERIFY=true`:

```bash
vault secrets tune -max-lease-ttl=87600h pki
vault write pki/root/generate/internal \
    common_name="starlabs.local CA" \
    ttl=87600h

vault write pki/config/urls \
    issuing_certificates="https://vault.starlabs.local:8200/v1/pki/ca" \
    crl_distribution_points="https://vault.starlabs.local:8200/v1/pki/crl"
```

The first command will return the certificate and the issuing CA. Save the certificate as `root-ca.pem`, this is your root CA. You can install this certificate in your OS/browser trust store to avoid warnings. In general Every machine / service that must trust your certificates needs this CA.

Examples:

#### Linux

```bash
sudo cp starlabs-root-ca.pem /usr/local/share/ca-certificates/starlabs-root-ca.crt
sudo update-ca-certificates
```

#### macOS

- Import into Keychain → System → Certificates → Always Trust

#### Windows

- Import into “Trusted Root Certification Authorities”

#### Docker containers

- Add to `/usr/local/share/ca-certificates`
- Run `update-ca-certificates`

#### Kubernetes nodes

- Add to host trust store
- Or inject into trust bundle if needed

### Issue a TLS certificate for Vault

Create a role for your domain:

```bash
vault write pki/roles/lab-dot-local \
    allowed_domains="starlabs.local" \
    allow_subdomains=true \
    max_ttl="720h"
```

Issue a certificate for `vault.starlabs.local`:

These commands must be executed outside the Vault container, in the Docker host or your admin machine. Instal `vault` CLI in your host and execute the `vault` commands from it. You can use the `jq` command to extract the values:

```bash
vault write -format=json pki/issue/lab-dot-local \
  common_name="vault.starlabs.local" \
  ip_sans="127.0.0.1" \
  > vault-starlabs-local.json
```

Save, using `jq`, the following values:

- `certificate`
- `private_key`
- `issuing_ca`

```bash
jq -r .data.certificate vault-starlabs-local.json > vault-starlabs-local.crt
jq -r .data.private_key vault-starlabs-local.json > vault-starlabs-local.key
jq -r .data.issuing_ca vault-starlabs-local.json > starlabs-ca.pem
```

### Replace Vault self-signed cert

Copy the cert and key inside the container:

```bash
cp vault-starlabs-local.crt /opt/vault/certs/vault.crt
cp vault-starlabs-local.key /opt/vault/certs/vault.key
```

Review and update the Vault container config if you need to:

```hcl
listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_cert_file = "/vault/certs/vault.crt"
  tls_key_file  = "/vault/certs/vault.key"
}
```

Restart the container:

```bash
docker restart vault
```

Remember to unseal Vault after the restart either manually or with the script we created before.

### Trust Vault CA everywhere

In the host:

```bash
sudo cp starlabs-ca.pem /usr/local/share/ca-certificates/vault-ca.crt
sudo update-ca-certificates
```

From your host or your admin machine test vault without `VAULT_SKIP_VERIFY`:

```bash
unset VAULT_SKIP_VERIFY
vault status
```

---

With this Vault is configured to issue certificates for your homelab services.

In the next article we will see how to integrate Vault with Traefik.

--Juanma
