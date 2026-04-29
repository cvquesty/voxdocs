# ⚙️ Configuration Reference

> *All the knobs, switches, and dials — because defaults are just suggestions.*

---

## Overview

OpenVox uses several configuration files to control the behavior of the agent, server, and related services. This guide covers all of them, with practical examples and explanations of the settings you'll actually use.

**Configuration files covered:**

- [`puppet.conf`](#puppetconf) — The main configuration file
- [`hiera.yaml`](#hierayaml) — Hiera hierarchy configuration
- [`r10k.yaml`](#r10kyaml) — Code deployment configuration
- [`puppetdb.conf`](#puppetdbconf) — PuppetDB connection settings
- [`fileserver.conf`](#fileserverconf) — Custom file serving mount points
- [Environment configuration](#environment-configuration)
- [Important directories](#important-directories)

---

## puppet.conf

The main configuration file for both the agent and server. Located at:

```text
/etc/puppetlabs/puppet/puppet.conf
```

### File Format

`puppet.conf` uses INI-style sections. Settings in `[main]` apply everywhere; settings in `[agent]` or `[server]` override `[main]` for that specific context.

```ini
[main]
# Settings that apply to both agent and server
certname = openvox.example.com
server = openvox.example.com
environment = production
vardir = /opt/puppetlabs/puppet/cache
logdir = /var/log/puppetlabs/puppet
rundir = /var/run/puppetlabs

[agent]
# Agent-specific settings
runinterval = 30m
report = true
pluginsync = true
show_diff = true

[server]
# Server-specific settings (only on the Primary Server)
dns_alt_names = puppet,openvox.example.com,openvox
environmentpath = /etc/puppetlabs/code/environments
storeconfigs = true
storeconfigs_backend = puppetdb
reports = store,puppetdb
```

### Essential Settings

#### `[main]` Section

| Setting | Default | Description |
|---------|---------|-------------|
| `certname` | Node's FQDN | The unique identifier for this node's SSL certificate. **Do not change after initial setup** unless you also clean/regenerate certs. |
| `server` | `puppet` | Hostname of the Primary Server. Agents connect here. |
| `environment` | `production` | The Puppet environment to use. Maps to a directory under `environmentpath`. |
| `vardir` | `/opt/puppetlabs/puppet/cache` | Where Puppet stores its runtime data (cached catalogs, facts, state). |
| `logdir` | `/var/log/puppetlabs/puppet` | Log file directory. |
| `ssldir` | `$vardir/ssl` | SSL certificate directory. |
| `codedir` | `/etc/puppetlabs/code` | Root of the code directory (environments, modules, Hiera data). |
| `environmentpath` | `$codedir/environments` | Where environment directories live. |
| `basemodulepath` | `$codedir/modules:/opt/puppetlabs/puppet/modules` | Module paths searched after the environment's own modules. |
| `hiera_config` | `$confdir/hiera.yaml` | Path to the global Hiera configuration. |

#### `[agent]` Section

| Setting | Default | Description |
|---------|---------|-------------|
| `runinterval` | `30m` | How often the agent runs (supports time suffixes: `s`, `m`, `h`, `d`). |
| `report` | `true` | Whether to send reports to the server. |
| `pluginsync` | `true` | Download plugins (custom facts, functions, types) from the server. |
| `show_diff` | `false` | Show file diffs in agent output. **Warning:** May expose sensitive data in logs! |
| `noop` | `false` | Default to noop mode (dry-run). Useful for new nodes. |
| `splay` | `false` | Add random delay before agent runs to spread load. |
| `splaylimit` | `$runinterval` | Maximum random delay when splay is enabled. |
| `usecacheonfailure` | `true` | Use cached catalog if the server is unreachable. |
| `waitforcert` | `120` | Seconds to wait for cert signing during first run. `0` to disable. |
| `daemonize` | `true` | Run agent as a background daemon. |

#### `[server]` Section

| Setting | Default | Description |
|---------|---------|-------------|
| `dns_alt_names` | — | Extra hostnames for the server's certificate (comma-separated). Set before initial setup! |
| `storeconfigs` | `false` | Enable exported resources via PuppetDB. |
| `storeconfigs_backend` | `puppetdb` | Backend for stored configurations. |
| `reports` | `store` | Comma-separated list of report processors. Common: `store,puppetdb`. |
| `ca` | `true` | Whether this server acts as the Certificate Authority. |
| `ca_ttl` | `5y` | Certificate TTL (how long certs are valid). |
| `autosign` | `false` | Auto-sign certificate requests. Can be `true`, `false`, or a path to an autosign script. |

### Modifying Settings

You can edit `puppet.conf` directly, or use the CLI:

```bash
# View a setting
puppet config print server

# Set a value
sudo puppet config set runinterval 1h --section agent

# Remove a setting (revert to default)
sudo puppet config delete show_diff --section agent

# Print ALL settings (hundreds of them!)
puppet config print all

# Print settings for a specific section
puppet config print all --section agent
```

### Example: Production Agent Configuration

```ini
[main]
certname = webserver1.example.com
server = openvox.example.com
environment = production

[agent]
runinterval = 30m
report = true
pluginsync = true
show_diff = false
usecacheonfailure = true
```

### Real-World Primary Server Configuration

Here's the **actual** `puppet.conf` from `openvox.example.com` — a live production OpenVox server managing a three-node fleet:

```ini
# /etc/puppetlabs/puppet/puppet.conf on openvox.example.com
[server]
node_terminus = exec
external_nodes = /opt/openvox-gui/scripts/enc.py
vardir = /opt/puppetlabs/server/data/puppetserver
logdir = /var/log/puppetlabs/puppetserver
rundir = /var/run/puppetlabs/puppetserver
pidfile = /var/run/puppetlabs/puppetserver/puppetserver.pid
codedir = /etc/puppetlabs/code

[agent]
server = openvox.example.com
certname = openvox.example.com
runinterval = 600
environment = production
number_of_facts_soft_limit = 8960

[main]
hiera_config = /etc/puppetlabs/puppet/hiera.yaml
environmentpath = $codedir/environments
basemodulepath = $confdir/modules:/opt/puppetlabs/puppet/modules
logdir = /var/log/puppetlabs/puppet/puppet_agent.log
rundir = /var/run/puppetlabs
ssldir = $confdir/ssl
server = openvox.example.com
log_level = err

[master]
storeconfigs = true
storeconfigs_backend = puppetdb
reports = store, puppetdb
```

> **Things to notice:**
>
> - The `[server]` section uses an **External Node Classifier** (`node_terminus = exec` + `external_nodes`) via the [OpenVox GUI](https://github.com/cvquesty/openvox-gui)
> - `runinterval = 600` means agents check in every 10 minutes (600 seconds)
> - `number_of_facts_soft_limit = 8960` — a tuning parameter for large fact sets
> - The `[master]` section name is a legacy alias for `[server]` — both work, but `[server]` is preferred in modern configurations

---

## hiera.yaml

The Hiera configuration file defines the lookup hierarchy for your data. There are **three layers** of Hiera configuration:

| Layer | Location | Purpose |
|-------|----------|---------|
| **Global** | `/etc/puppetlabs/puppet/hiera.yaml` | System-wide defaults |
| **Environment** | `<environmentpath>/<env>/hiera.yaml` | Per-environment hierarchy |
| **Module** | `<module>/hiera.yaml` | Module-specific data |

> **Note:** For a deep dive into Hiera, including merge strategies, eyaml encryption, and advanced hierarchies, see the [Hiera Deep-Dive](../hiera/README.md).

### Example: Environment hiera.yaml

```yaml
---
version: 5

defaults:
  datadir: data
  data_hash: yaml_data

hierarchy:
  - name: "Per-node data"
    path: "nodes/%{trusted.certname}.yaml"

  - name: "Per-OS data"
    path: "os/%{facts.os.family}.yaml"

  - name: "Per-environment data"
    path: "environment/%{environment}.yaml"

  - name: "Common defaults"
    path: "common.yaml"
```

### Hierarchy Variables

You can interpolate these variables in hierarchy paths:

| Variable | Example Value | Source |
|----------|---------------|--------|
| `%{trusted.certname}` | `web01.example.com` | Certificate (tamper-proof) |
| `%{facts.os.family}` | `RedHat` | Facter |
| `%{facts.os.name}` | `Rocky` | Facter |
| `%{facts.networking.fqdn}` | `web01.example.com` | Facter |
| `%{environment}` | `production` | Current environment |
| `%{facts.virtual}` | `kvm` | Facter |

---

## r10k.yaml

Controls how r10k deploys code from Git. Located at:

```text
/etc/puppetlabs/r10k/r10k.yaml
```

### Example Configuration

```yaml
---
# Git sources — each source maps branches to environments
sources:
  control:
    remote: 'git@github.com:myorg/control-repo.git'
    basedir: '/etc/puppetlabs/code/environments'
    prefix: false     # Don't prefix environment names with the source name

# Git cache directory
cachedir: '/opt/puppetlabs/puppet/cache/r10k'

# Forge settings (for Puppetfile module installation)
forge:
  baseurl: 'https://forgeapi.puppet.com'

# Post-deployment hooks (optional)
# postrun:
#   - '/usr/local/bin/refresh-environments.sh'

# Git settings
git:
  provider: rugged    # Use the rugged gem for Git operations
  # private_key: '/etc/puppetlabs/r10k/id_rsa'   # Deploy key for private repos
```

### Control Repository Structure

r10k expects your control repo to have this structure:

```text
control-repo/
├── Puppetfile           ← Module dependencies (like a Gemfile for Puppet)
├── environment.conf     ← Environment-specific settings
├── hiera.yaml           ← Hiera hierarchy for this environment
├── manifests/
│   └── site.pp          ← Main site manifest
├── data/
│   ├── common.yaml
│   └── nodes/
├── site-modules/        ← Custom modules specific to your org
│   ├── role/
│   └── profile/
└── modules/             ← (managed by r10k — don't edit!)
```

### Puppetfile Example

```ruby
# Puppetfile — declares module dependencies

# Modules from the Puppet Forge
mod 'puppetlabs-stdlib',     '9.6.0'
mod 'puppetlabs-apache',     '12.1.0'
mod 'puppetlabs-mysql',      '15.0.0'
mod 'puppetlabs-firewall',   '8.0.3'
mod 'puppet-nginx',          '6.0.1'

# Module from a Git repo
mod 'custom_facts',
  git:    'git@github.com:myorg/custom_facts.git',
  branch: 'main'

# Module from a specific tag
mod 'internal_profiles',
  git: 'git@github.com:myorg/internal_profiles.git',
  tag: 'v2.1.0'
```

---

## puppetdb.conf

Tells the Puppet agent where to find PuppetDB. Located at:

```text
/etc/puppetlabs/puppet/puppetdb.conf
```

### Example Configuration

```ini
[main]
server_urls = https://openvox.example.com:8081
```

For the server's PuppetDB route configuration:

```text
/etc/puppetlabs/puppet/routes.yaml
```

```yaml
---
server:
  facts:
    terminus: puppetdb
    cache: yaml
```

### PuppetDB Server Configuration

PuppetDB's own configuration lives in:

```text
/etc/puppetlabs/puppetdb/conf.d/
```

Key files:

| File | Purpose |
|------|---------|
| `database.ini` | PostgreSQL connection string |
| `jetty.ini` | Network listener configuration (port 8081) |
| `config.ini` | General PuppetDB settings |

#### `database.ini`

```ini
[database]
classname = org.postgresql.Driver
subprotocol = postgresql
subname = //localhost:5432/puppetdb
username = puppetdb
password = your_password_here
```

#### `jetty.ini`

```ini
[jetty]
ssl-host = 0.0.0.0
ssl-port = 8081
ssl-cert = /etc/puppetlabs/puppetdb/ssl/public.pem
ssl-key = /etc/puppetlabs/puppetdb/ssl/private.pem
ssl-ca-cert = /etc/puppetlabs/puppet/ssl/certs/ca.pem
```

---

## fileserver.conf

Configure custom file server mount points for serving files to agents. Located at:

```text
/etc/puppetlabs/puppet/fileserver.conf
```

```ini
# Serve files from a custom path
[extra_files]
  path /opt/puppet-files
  allow *

# Restrict access by certname
[secret_data]
  path /opt/secret-data
  allow *.internal.example.com
  deny  *.external.example.com
```

Access in manifests:

```puppet
file { '/etc/app.conf':
  source => 'puppet:///extra_files/app.conf',
}
```

---

## Environment Configuration

Each environment can have its own `environment.conf`:

```text
/etc/puppetlabs/code/environments/production/environment.conf
```

```ini
# Override the module path for this environment
modulepath = site-modules:modules:$basemodulepath

# Set a custom manifest
manifest = manifests/site.pp

# Set config version (appears in reports)
config_version = '/usr/bin/git --git-dir /etc/puppetlabs/code/environments/production/.git rev-parse HEAD'

# Environment timeout (how long to cache the environment)
# 0 = reload every request (good for development)
# unlimited = cache forever (good for production)
environment_timeout = unlimited
```

> **Important:** In production, set `environment_timeout = unlimited` for performance. This means you must restart PuppetServer (or use the environment cache API) after deploying new code. r10k handles this automatically.

---

## Important Directories

Here's a map of where everything lives on disk:

```text
/etc/puppetlabs/
├── puppet/
│   ├── puppet.conf              ← Main config
│   ├── hiera.yaml               ← Global Hiera config
│   ├── puppetdb.conf            ← PuppetDB connection
│   ├── routes.yaml              ← Data routing (facts → PuppetDB)
│   ├── fileserver.conf          ← Custom file mounts
│   ├── autosign.conf            ← Autosign rules (if used)
│   ├── csr_attributes.yaml      ← CSR extension attributes
│   └── ssl/                     ← SSL certificates and keys
│       ├── certs/
│       │   ├── ca.pem           ← CA certificate
│       │   └── <certname>.pem   ← Node certificate
│       ├── private_keys/
│       │   └── <certname>.pem   ← Node private key
│       └── certificate_requests/
│
├── puppetserver/
│   └── conf.d/
│       ├── puppetserver.conf    ← JVM and server settings
│       ├── webserver.conf       ← Jetty web server config
│       ├── web-routes.conf      ← URL routing
│       ├── ca.conf              ← CA configuration
│       ├── auth.conf            ← Authorization rules
│       └── metrics.conf         ← Metrics/monitoring
│
├── puppetdb/
│   └── conf.d/
│       ├── database.ini         ← PostgreSQL connection
│       ├── jetty.ini            ← Network listener
│       └── config.ini           ← General settings
│
├── code/
│   └── environments/
│       ├── production/
│       │   ├── manifests/site.pp
│       │   ├── modules/
│       │   ├── site-modules/
│       │   ├── data/
│       │   ├── hiera.yaml
│       │   ├── environment.conf
│       │   └── Puppetfile
│       └── staging/
│           └── ...
│
├── r10k/
│   └── r10k.yaml               ← r10k configuration
│
└── bolt/
    ├── bolt-project.yaml        ← Bolt project config
    └── inventory.yaml           ← Bolt inventory
```

### Log Files

| Service | Log Location |
|---------|-------------|
| Puppet Agent | `/var/log/puppetlabs/puppet/puppet.log` |
| PuppetServer | `/var/log/puppetlabs/puppetserver/puppetserver.log` |
| PuppetServer Access | `/var/log/puppetlabs/puppetserver/puppetserver-access.log` |
| PuppetDB | `/var/log/puppetlabs/puppetdb/puppetdb.log` |
| r10k | `/var/log/puppetlabs/r10k/r10k.log` (if configured) |

### systemd Service Names

| Service | Unit Name |
|---------|-----------|
| Puppet Agent | `puppet` |
| PuppetServer | `puppetserver` (or `openvox-server`) |
| PuppetDB | `puppetdb` |

```bash
# Manage services
sudo systemctl status puppet
sudo systemctl restart puppetserver
sudo systemctl enable puppetdb

# View logs
sudo journalctl -u puppetserver -f
sudo journalctl -u puppet --since "1 hour ago"
```

---

*Next up: [Server Administration →](../server-admin/README.md)*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
