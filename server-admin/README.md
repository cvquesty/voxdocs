# 🖥️ Server Administration Guide

> *Keeping the lights on, the certs signed, and the catalogs compiled.*

---

> **Sources:** Service names, package names, and prerequisites (JDK, PostgreSQL)
> align with [docs.openvoxproject.org](https://docs.openvoxproject.org/)
> (OpenVox server + OpenVoxDB sections). Tuning tables and lab procedures are
> community color — [EDITORIAL.md](../EDITORIAL.md).

---

## Overview

Running an OpenVox infrastructure involves three server-side roles:

| Role (official name) | Package | systemd unit / binary | Notes |
|----------------------|---------|------------------------|--------|
| **OpenVox server** | `openvox-server` | `puppetserver` | Catalog compiler; usually hosts the CA |
| **OpenVoxDB** | `openvoxdb` + `openvoxdb-termini` | `puppetdb` | Facts, catalogs, reports, exports |
| **CA** | (inside OpenVox server by default) | `puppetserver ca …` | Can be split in advanced topologies |

**Official facts that bite people in production:**

- OpenVox server needs a supported **JDK 17 or 21** installed **separately** (not inside the RPM/DEB).
- OpenVoxDB needs **PostgreSQL 11+** (project recommends **14+**), also external.
- **r10k is not** shipped with `openvox-server` — install it yourself or use **OpenBolt**’s bundled r10k for orchestration workflows.

This guide is day-to-day admin, tuning, backup, and “why is the heap on fire?”

---

## PuppetServer Administration

### Starting and Stopping

```bash
# Start / stop / restart
sudo systemctl start puppetserver
sudo systemctl stop puppetserver
sudo systemctl restart puppetserver

# Graceful reload (reloads config without dropping connections)
sudo puppetserver reload

# Check status
sudo systemctl status puppetserver

# View logs in real-time
sudo journalctl -u puppetserver -f

# Start in foreground (useful for debugging)
sudo puppetserver foreground
```

### JVM Memory Tuning

PuppetServer runs on the JVM, and memory is the #1 performance knob. Edit:

```text
/etc/puppetlabs/puppetserver/conf.d/puppetserver.conf
```

Look for the `jruby-puppet` section:

```hocon
jruby-puppet: {
    # Number of JRuby instances (roughly: one per CPU core, minus 1)
    max-active-instances: 3

    # Maximum number of requests per JRuby before recycling
    max-requests-per-instance: 100000
}
```

And the JVM heap in `/etc/sysconfig/puppetserver` (RHEL) or `/etc/default/puppetserver` (Debian):

```bash
# JVM heap — recommended: 2GB per JRuby instance + 512MB overhead
# For 3 JRuby instances: 3 * 2GB + 512MB ≈ 7GB
JAVA_ARGS="-Xms4g -Xmx4g -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"
```

**Rules of thumb:**

| Fleet Size | JRuby Instances | JVM Heap |
|-----------|-----------------|----------|
| < 100 nodes | 1-2 | 2-3 GB |
| 100-500 nodes | 2-4 | 4-6 GB |
| 500-2000 nodes | 4-8 | 6-12 GB |
| 2000+ nodes | 8+ (consider compile servers) | 12+ GB |

> **Warning:** Setting the heap too large can cause long GC pauses. Monitor with the metrics endpoint and adjust accordingly. Also, don't set `-Xms` different from `-Xmx` — let the JVM start with the full heap.

### Environment Cache

By default, PuppetServer caches parsed environments. After deploying new code with r10k, you need to flush the cache:

```bash
# Flush the environment cache via the admin API
curl -i --cert /etc/puppetlabs/puppet/ssl/certs/$(puppet config print certname).pem \
     --key /etc/puppetlabs/puppet/ssl/private_keys/$(puppet config print certname).pem \
     --cacert /etc/puppetlabs/puppet/ssl/certs/ca.pem \
     -X DELETE \
     https://$(puppet config print certname):8140/puppet-admin-api/v1/environment-cache

# Or just restart the server (simpler but causes brief downtime)
sudo systemctl restart puppetserver
```

> **Pro tip:** r10k's `postrun` hook can automatically flush the cache after every deployment.

---

## OpenVoxDB (PuppetDB) Administration

### Installation

**OpenVoxDB** (the rebranded PuppetDB) **requires** PostgreSQL 11 or later, but the official OpenVox project now **recommends PostgreSQL 14 or later** to take advantage of newer query-planner and indexing features. The packages are `openvoxdb` and `openvoxdb-termini`, which provide the `puppetdb` systemd service and binaries for compatibility. If your distribution doesn't ship a recent enough PostgreSQL, install one from the [PostgreSQL Global Development Group](https://yum.postgresql.org/) (yum) or [apt.postgresql.org](https://apt.postgresql.org/) (apt) repositories.

```bash
# Install PostgreSQL
sudo yum install -y postgresql-server postgresql-contrib
sudo postgresql-setup --initdb
sudo systemctl enable --now postgresql

# Create the PuppetDB database and user
sudo -u postgres createuser -DRSP puppetdb
sudo -u postgres createdb -O puppetdb puppetdb

# Install OpenVoxDB (package provides puppetdb service for compatibility)
sudo yum install -y openvoxdb openvoxdb-termini

# Configure SSL
sudo puppetdb ssl-setup

# Start PuppetDB
sudo systemctl enable --now puppetdb
```

### Useful PQL Queries

```bash
# How many nodes are in my fleet?
puppet query 'nodes[count()] {}'

# Which nodes haven't checked in recently?
puppet query 'nodes[certname, report_timestamp] { report_timestamp < "1 hour ago" }'

# What OS versions are in use?
puppet query 'facts[certname, value] { name = "os.release.full" }'

# Find all nodes with a specific class
puppet query 'resources[certname] { type = "Class" and title = "Apache" }'

# Find all failed nodes from last run
puppet query 'nodes[certname] { latest_report_status = "failed" }'

# Find nodes with a specific fact value
puppet query 'inventory[certname] { facts.os.name = "Rocky" and facts.os.release.major = "9" }'
```

### Database Maintenance

```bash
# Vacuum and analyze the PuppetDB database
sudo -u postgres vacuumdb --analyze puppetdb

# Check database size
sudo -u postgres psql puppetdb -c "SELECT pg_size_pretty(pg_database_size('puppetdb'));"

# Configure automatic garbage collection in config.ini
# /etc/puppetlabs/puppetdb/conf.d/config.ini
# [database]
# gc-interval = 60         # Minutes between GC runs
# node-ttl = 14d           # Remove nodes inactive for 14 days
# node-purge-ttl = 14d     # Purge node data after 14 days
# report-ttl = 30d         # Keep reports for 30 days
```

---

## Certificate Authority Management

### Day-to-Day Operations

```bash
# List pending certificate requests
sudo puppetserver ca list

# List ALL certificates (signed + pending)
sudo puppetserver ca list --all

# Sign a new node's certificate
sudo puppetserver ca sign --certname newnode.example.com

# Sign all pending requests (use with caution!)
sudo puppetserver ca sign --all

# Revoke a compromised or decommissioned node
sudo puppetserver ca revoke --certname badnode.example.com

# Clean (remove completely) a certificate
sudo puppetserver ca clean --certname oldnode.example.com
```

### Re-registering a Node

If a node needs to be re-registered (new hardware, rebuilt, etc.):

**On the server:**

```bash
sudo puppetserver ca clean --certname node.example.com
```

**On the node:**

```bash
sudo puppet ssl clean
sudo puppet agent -t  # This will generate a new CSR
```

**Back on the server:**

```bash
sudo puppetserver ca sign --certname node.example.com
```

### Autosigning (Use With Caution!)

For automated provisioning, you can enable autosigning. There are several approaches:

#### Basic Autosign (all requests — DANGEROUS in production)

```ini
# In puppet.conf [server] section
autosign = true
```

#### Allowlist-based Autosign

```text
# /etc/puppetlabs/puppet/autosign.conf
*.dev.example.com
staging-*.example.com
```

#### Script-based Autosign (recommended for automation)

```bash
#!/bin/bash
# /etc/puppetlabs/puppet/autosign.sh
# This script receives the certname as $1 and CSR on stdin
# Exit 0 to sign, non-zero to reject

CERTNAME="$1"

# Only autosign if the certname matches our pattern
if [[ "$CERTNAME" =~ ^auto-[a-z0-9]+-[0-9]+\.example\.com$ ]]; then
    exit 0
fi

exit 1
```

```ini
# In puppet.conf
autosign = /etc/puppetlabs/puppet/autosign.sh
```

---

## Backup and Recovery

### What to Back Up

| Component | Path | Priority |
|-----------|------|----------|
| **CA certificates** | `/etc/puppetlabs/puppet/ssl/` | 🔴 CRITICAL — lose this and ALL nodes need new certs |
| **PuppetDB database** | PostgreSQL dump | 🟡 Important — historical data, reports |
| **Configuration** | `/etc/puppetlabs/` | 🟡 Important — can be rebuilt but tedious |
| **Code** | Git repository (control repo) | 🟢 Already in Git (you ARE using Git, right?) |

### Backup Script

```bash
#!/bin/bash
# backup-openvox.sh — Back up critical OpenVox data
BACKUP_DIR="/opt/backups/openvox/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# 1. Back up the CA (CRITICAL!)
sudo tar czf "$BACKUP_DIR/puppet-ssl.tar.gz" \
  /etc/puppetlabs/puppet/ssl/

# 2. Back up configuration
sudo tar czf "$BACKUP_DIR/puppet-config.tar.gz" \
  /etc/puppetlabs/puppet/puppet.conf \
  /etc/puppetlabs/puppet/hiera.yaml \
  /etc/puppetlabs/puppet/puppetdb.conf \
  /etc/puppetlabs/puppetserver/conf.d/ \
  /etc/puppetlabs/puppetdb/conf.d/ \
  /etc/puppetlabs/r10k/

# 3. Back up PuppetDB
sudo -u postgres pg_dump puppetdb | gzip > "$BACKUP_DIR/puppetdb.sql.gz"

echo "Backup complete: $BACKUP_DIR"
```

### Recovery

```bash
# Restore CA certificates
sudo tar xzf puppet-ssl.tar.gz -C /

# Restore PuppetDB
sudo -u postgres dropdb puppetdb
sudo -u postgres createdb -O puppetdb puppetdb
gunzip -c puppetdb.sql.gz | sudo -u postgres psql puppetdb

# Redeploy code
sudo r10k deploy environment --verbose

# Restart all services
sudo systemctl restart puppetserver puppetdb puppet
```

---

## Monitoring and Health Checks

### PuppetServer Status

```bash
# Check the server status API
curl -s https://localhost:8140/status/v1/services \
  --cert /etc/puppetlabs/puppet/ssl/certs/$(hostname -f).pem \
  --key /etc/puppetlabs/puppet/ssl/private_keys/$(hostname -f).pem \
  --cacert /etc/puppetlabs/puppet/ssl/certs/ca.pem | python3 -m json.tool
```

### PuppetDB Status

```bash
# Check PuppetDB status
curl -s https://localhost:8081/status/v1/services \
  --cert /etc/puppetlabs/puppetdb/ssl/public.pem \
  --key /etc/puppetlabs/puppetdb/ssl/private.pem \
  --cacert /etc/puppetlabs/puppet/ssl/certs/ca.pem | python3 -m json.tool

# Check node count
puppet query 'nodes[count()] {}'

# Check for stale nodes (no check-in for 2 hours)
puppet query 'nodes[certname, report_timestamp] { report_timestamp < "2 hours ago" }'
```

### Quick Health Check Script

```bash
#!/bin/bash
# health-check.sh — Quick OpenVox infrastructure health check

echo "=== OpenVox Health Check ==="
echo ""

# Service status
for svc in puppetserver puppetdb puppet; do
  STATUS=$(systemctl is-active $svc 2>/dev/null)
  if [ "$STATUS" = "active" ]; then
    echo "✅ $svc: running"
  else
    echo "❌ $svc: $STATUS"
  fi
done

echo ""

# Node statistics
TOTAL=$(puppet query 'nodes[count()] {}' 2>/dev/null | grep -o '[0-9]*')
CHANGED=$(puppet query 'nodes[count()] { latest_report_status = "changed" }' 2>/dev/null | grep -o '[0-9]*')
FAILED=$(puppet query 'nodes[count()] { latest_report_status = "failed" }' 2>/dev/null | grep -o '[0-9]*')

echo "📊 Fleet: ${TOTAL:-?} total, ${CHANGED:-?} changed, ${FAILED:-?} failed"

# Disk usage
echo ""
echo "💾 Disk usage:"
df -h /etc/puppetlabs /opt/puppetlabs 2>/dev/null | tail -n +2
```

---

## Scaling Tips

- **Compile servers**: For large fleets (1000+ nodes), add additional PuppetServer instances that compile catalogs but don't run the CA
- **PostgreSQL tuning**: Increase `shared_buffers`, `work_mem`, and `effective_cache_size` for PuppetDB performance
- **r10k caching**: Use `cachedir` to avoid re-cloning Git repos on every deployment
- **Agent splay**: Enable `splay = true` in `puppet.conf` to spread agent check-ins over the run interval
- **Environment caching**: Set `environment_timeout = unlimited` in production for faster compilations

---

*Next up: [Hiera Deep-Dive →](../hiera/README.md)*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
