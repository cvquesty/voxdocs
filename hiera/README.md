# 📚 Hiera Deep-Dive

> *Separating data from code — because hardcoding passwords in manifests is a career-limiting move.*

---

## What Is Hiera?

Hiera (from "hierarchy") is Puppet's built-in **hierarchical data lookup system**. It lets you store configuration data — ports, passwords, hostnames, feature flags, anything — in YAML files, organized in a priority hierarchy. Your Puppet code then **looks up** these values at compile time, keeping your manifests clean and your data organized.

Think of it this way:
- **Puppet code** (manifests, modules) = **logic** ("install Apache, configure it, start the service")
- **Hiera data** (YAML files) = **values** ("the port is 8080, the SSL cert is here, use these modules")

This separation means the same code can work across different nodes, environments, and datacenters — you just change the data.

---

## The Three Layers

Hiera has three layers of configuration, which combine into a single ordered "super-hierarchy":

```
┌─────────────────────────────────────────────────┐
│  1. Global Layer                                │
│     /etc/puppetlabs/puppet/hiera.yaml           │
│     (applies to all environments)               │
├─────────────────────────────────────────────────┤
│  2. Environment Layer                           │
│     <env>/hiera.yaml                            │
│     (per-environment hierarchy)                 │
├─────────────────────────────────────────────────┤
│  3. Module Layer                                │
│     <module>/hiera.yaml                         │
│     (module-specific defaults)                  │
└─────────────────────────────────────────────────┘
```

When Puppet looks up a key, it searches **all three layers** in order. The environment layer is where most of your action happens.

---

## Writing hiera.yaml

### Version 5 Format

```yaml
---
version: 5

defaults:
  datadir: data           # Relative to the environment/module root
  data_hash: yaml_data    # Backend: yaml_data, json_data, or custom

hierarchy:
  - name: "Per-node data"
    path: "nodes/%{trusted.certname}.yaml"

  - name: "Per-datacenter"
    path: "datacenter/%{facts.networking.domain}.yaml"

  - name: "Per-OS family"
    path: "os/%{facts.os.family}.yaml"

  - name: "Per-environment"
    path: "env/%{environment}.yaml"

  - name: "Common defaults"
    path: "common.yaml"
```

**How it works:** When looking up a key (say, `ntp::servers`), Hiera walks the hierarchy top-to-bottom:

1. Check `data/nodes/web01.example.com.yaml` — found? Return it!
2. Not found → check `data/datacenter/example.com.yaml`
3. Not found → check `data/os/RedHat.yaml`
4. Not found → check `data/env/production.yaml`
5. Not found → check `data/common.yaml`
6. Not found → return `undef` (or the default value)

### Using `paths` (Multiple Paths per Level)

```yaml
hierarchy:
  - name: "Virtual-specific data"
    paths:
      - "virtual/%{facts.virtual}.yaml"
      - "virtual/physical.yaml"        # Fallback for bare metal
```

### Using `glob` (Wildcard Patterns)

```yaml
hierarchy:
  - name: "Site configs"
    glob: "sites/*.yaml"    # Loads ALL matching files
```

---

## Data Files

Data files are plain YAML, with keys that match class parameters:

### `data/common.yaml`

```yaml
---
# NTP configuration
ntp::servers:
  - '0.pool.ntp.org'
  - '1.pool.ntp.org'
  - '2.pool.ntp.org'

# SSH configuration
ssh::permit_root_login: 'no'
ssh::password_authentication: 'no'
ssh::port: 22

# Base packages to install everywhere
profile::base::packages:
  - vim
  - curl
  - wget
  - tree
  - htop

# DNS resolvers
resolv_conf::nameservers:
  - '8.8.8.8'
  - '8.8.4.4'
```

### `data/os/RedHat.yaml`

```yaml
---
# RedHat-specific overrides
ntp::package_name: 'chrony'
ntp::service_name: 'chronyd'

ssh::service_name: 'sshd'
```

### `data/os/Debian.yaml`

```yaml
---
# Debian-specific overrides
ntp::package_name: 'chrony'
ntp::service_name: 'chrony'

ssh::service_name: 'ssh'
```

### `data/nodes/webserver1.example.com.yaml`

```yaml
---
# Node-specific overrides for webserver1
ntp::servers:
  - '10.0.0.1'   # Internal NTP server
  - '10.0.0.2'

apache::port: 8080
apache::ssl: true

profile::base::extra_packages:
  - mod_ssl
  - php
```

---

## Automatic Parameter Lookup

The best part of Hiera: **you don't even need to call `lookup()` explicitly**. When a class declares parameters, Puppet automatically looks them up in Hiera using the pattern `classname::parameter`:

```puppet
# This class...
class ntp (
  Array[String] $servers,
  String        $package_name = 'ntp',
  String        $service_name = 'ntpd',
) {
  # ...
}
```

```yaml
# ...is automatically populated by this Hiera data:
ntp::servers:
  - '0.pool.ntp.org'
  - '1.pool.ntp.org'
ntp::package_name: 'chrony'
ntp::service_name: 'chronyd'
```

Just `include ntp` in your manifest, and Hiera fills in the parameters. No `class { 'ntp': servers => [...] }` needed! This is the recommended pattern for modern Puppet/OpenVox.

---

## Merge Strategies

By default, Hiera returns the **first** value it finds. But for arrays and hashes, you often want to **merge** values from multiple hierarchy levels.

### The Four Merge Strategies

| Strategy | Behavior | Best For |
|----------|----------|----------|
| `first` | Return first match (default) | Simple values, strings, booleans |
| `unique` | Merge arrays, remove duplicates | Lists of packages, users, groups |
| `hash` | Shallow merge of hashes (higher levels win) | Simple config hashes |
| `deep` | Recursive deep merge of hashes | Complex nested configurations |

### Configuring Merges with `lookup_options`

Put this in any data file to control how specific keys merge:

```yaml
# data/common.yaml
---
lookup_options:
  # Deep merge all profile::base config
  profile::base::config:
    merge: deep

  # Unique merge for package lists
  profile::base::packages:
    merge: unique

  # Apply to keys matching a regex
  "^profile::.*::users$":
    merge: unique
```

### Deep Merge Example

```yaml
# data/common.yaml
myapp::config:
  database:
    host: 'localhost'
    port: 5432
  logging:
    level: 'info'
    file: '/var/log/myapp.log'

# data/nodes/prod-db1.example.com.yaml
myapp::config:
  database:
    host: 'db-primary.example.com'    # Override the host
    pool_size: 20                      # Add a new key
  cache:
    enabled: true                      # Add new section
```

**Result with `merge: deep`:**

```yaml
myapp::config:
  database:
    host: 'db-primary.example.com'    # From node data (override)
    port: 5432                         # From common (preserved)
    pool_size: 20                      # From node data (new)
  logging:
    level: 'info'                      # From common (preserved)
    file: '/var/log/myapp.log'         # From common (preserved)
  cache:
    enabled: true                      # From node data (new)
```

### Knock-out Prefix

Remove specific items from merged arrays/hashes:

```yaml
# data/common.yaml
profile::base::packages:
  - vim
  - curl
  - telnet
  - ftp

# data/os/SecureOS.yaml
profile::base::packages:
  - '--telnet'    # REMOVE telnet from the merged list
  - '--ftp'       # REMOVE ftp too

lookup_options:
  profile::base::packages:
    merge: unique
    knock_out_prefix: '--'
```

Result: `['vim', 'curl']`

---

## Encrypted Data with eyaml

Hiera-eyaml lets you **encrypt sensitive values** (passwords, API keys, private keys) in your data files while keeping the rest in plaintext. This means you can safely commit Hiera data to Git!

### Setup

```bash
# Install the eyaml gem
sudo /opt/puppetlabs/puppet/bin/gem install hiera-eyaml

# Generate encryption keys
/opt/puppetlabs/puppet/bin/eyaml createkeys

# Move keys to a secure location
sudo mkdir -p /etc/puppetlabs/puppet/eyaml
sudo mv ./keys/private_key.pkcs7.pem /etc/puppetlabs/puppet/eyaml/
sudo mv ./keys/public_key.pkcs7.pem /etc/puppetlabs/puppet/eyaml/
sudo chown puppet:puppet /etc/puppetlabs/puppet/eyaml/*.pem
sudo chmod 400 /etc/puppetlabs/puppet/eyaml/private_key.pkcs7.pem
```

### Configure hiera.yaml

```yaml
---
version: 5

defaults:
  datadir: data
  data_hash: yaml_data

hierarchy:
  - name: "Encrypted secrets"
    lookup_key: eyaml_lookup_key    # Use eyaml backend!
    path: "secrets.eyaml"
    options:
      pkcs7_private_key: /etc/puppetlabs/puppet/eyaml/private_key.pkcs7.pem
      pkcs7_public_key: /etc/puppetlabs/puppet/eyaml/public_key.pkcs7.pem

  - name: "Per-node data"
    path: "nodes/%{trusted.certname}.yaml"

  - name: "Common defaults"
    path: "common.yaml"
```

### Encrypt and Use Values

```bash
# Encrypt a value
/opt/puppetlabs/puppet/bin/eyaml encrypt -s 'SuperSecretP@ssw0rd!'

# Output:
# ENC[PKCS7,MIIBiQYJKoZIhvcNAQcDoIIBejCC...]

# Edit an eyaml file interactively
/opt/puppetlabs/puppet/bin/eyaml edit data/secrets.eyaml
```

```yaml
# data/secrets.eyaml
---
myapp::db_password: ENC[PKCS7,MIIBiQYJKoZIhvcNAQcDoIIBejCC...]
myapp::api_key: ENC[PKCS7,AnotherEncryptedValueHere...]
```

Puppet decrypts these values transparently at compile time. The encrypted values are safe to commit to Git — only the server with the private key can decrypt them.

---

## Debugging Lookups

### The `puppet lookup` Command

```bash
# Simple lookup
puppet lookup ntp::servers

# See WHERE the value came from (the money command!)
puppet lookup ntp::servers --explain

# Lookup with a specific merge strategy
puppet lookup profile::base::packages --merge unique --explain

# Lookup for a specific node
puppet lookup apache::port --node webserver1.example.com

# Lookup in a specific environment
puppet lookup myapp::config --environment staging

# Output as JSON
puppet lookup myapp::all_settings --render-as json
```

### Example `--explain` Output

```
$ puppet lookup ntp::servers --explain --environment production

Searching for "ntp::servers"
  Environment Data Provider (hiera configuration version 5)
    Using configuration "/etc/puppetlabs/code/environments/production/hiera.yaml"

    Hierarchy entry "Per-node data"
      Path "/etc/puppetlabs/code/environments/production/data/nodes/openvox.example.com.yaml"
        Original path: "nodes/%{trusted.certname}.yaml"
        Path not found

    Hierarchy entry "Per-OS family"
      Path "/etc/puppetlabs/code/environments/production/data/os/RedHat.yaml"
        Original path: "os/%{facts.os.family}.yaml"
        No such key: "ntp::servers"

    Hierarchy entry "Common defaults"
      Path "/etc/puppetlabs/code/environments/production/data/common.yaml"
        Original path: "common.yaml"
        Found key: "ntp::servers" value: ["0.pool.ntp.org", "1.pool.ntp.org", "2.pool.ntp.org"]
```

---

## Best Practices

1. **Use `trusted.certname` instead of `facts.networking.fqdn`** — trusted facts come from the certificate and can't be spoofed by a compromised agent

2. **Put secrets in eyaml** — never commit plaintext passwords to Git

3. **Use `lookup_options` for merge behavior** — define merge strategies in your data, not your code

4. **Keep `common.yaml` thin** — it should contain sensible defaults, not every parameter for every class

5. **Prefer automatic parameter lookup** over explicit `lookup()` calls — it's cleaner and easier to maintain

6. **Use the environment layer** for most hierarchies — the global layer is for cross-environment data only

7. **Test your lookups** — use `puppet lookup --explain` regularly, especially after changing the hierarchy

8. **Name data files by their interpolation** — if the path is `os/%{facts.os.family}.yaml`, create files named `RedHat.yaml`, `Debian.yaml`, etc.

9. **Use `puppet lookup` from the CLI** — quickly test lookups without running a full Puppet apply:
   ```bash
   sudo /opt/puppetlabs/puppet/bin/puppet lookup ntp::servers --environment production
   ```

10. **Keep YAML files valid** — use a linter (like `yamllint`) and always include the `---` document start marker. Trailing commas are YAML syntax errors, unlike Puppet manifests!

---

*Next up: [Module Development →](../module-development/README.md)*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
