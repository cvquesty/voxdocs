# 🔄 Migrating from Puppet 7 to OpenVox 8

> *Everything you need to know to make the jump — without breaking your infrastructure.*

---

> **Sources:** Migration paths and breaking changes should match official OpenVox
> release notes. Lab checklists remain community color — [EDITORIAL.md](../EDITORIAL.md).

---

## Overview

OpenVox 8 is based on **Puppet 8**, which includes several breaking changes from Puppet 7. If you're running Puppet 7 (or earlier) and planning to migrate to OpenVox, this guide covers what you need to know.

**The good news:** Most well-written Puppet code will work without modification. The breaking changes primarily affect legacy patterns that were already discouraged.

> **Two migration paths.** Per the official OpenVox release notes, you have two reasonable routes from open-source Puppet 7 to OpenVox 8:
>
> 1. **Puppet 7 → OpenVox 7 → OpenVox 8** — switch ecosystems first, then take the language jump. Lower risk because each step changes only one variable.
> 2. **Puppet 7 → Puppet 8 → OpenVox 8** — take the language jump first on the platform you know, then swap packages. Easier if your team is more comfortable debugging Puppet itself than the package transition.
>
> Either way works. Pick whichever your operational team finds less risky.

---

## Breaking Changes

### 1. Strict Mode is Now Default

In Puppet 8 / OpenVox 8, **`strict_variables` is enabled by default**. This means:

- Referencing an undefined variable causes a **compilation failure** (not just a warning)
- You can no longer rely on undefined variables evaluating to `undef`

**Before (Puppet 7 — worked but risky):**

```puppet
# This would silently return undef if $myvar wasn't defined
if $myvar {
  notify { 'Variable is set': }
}
```

**After (Puppet 8 / OpenVox 8 — must be explicit):**

```puppet
# Option 1: Use a default value
$myvar = lookup('myapp::myvar', String, 'first', undef)
if $myvar {
  notify { 'Variable is set': }
}

# Option 2: Check if variable is defined
if defined('$myvar') and $myvar {
  notify { 'Variable is set': }
}

# Option 3: Use pick() with a fallback
$safe_var = pick($myvar, 'default_value')
```

**To check your code:** Run `puppet parser validate` on all manifests. Undefined variable references will now be flagged as errors.

---

### 2. Legacy Facts are Removed

**All legacy top-scope facts have been removed.** You must use the structured `$facts` hash.

| ❌ Legacy (Removed) | ✅ Structured (Required) |
|---------------------|--------------------------|
| `$osfamily` | `$facts['os']['family']` |
| `$operatingsystem` | `$facts['os']['name']` |
| `$operatingsystemrelease` | `$facts['os']['release']['full']` |
| `$operatingsystemmajrelease` | `$facts['os']['release']['major']` |
| `$fqdn` | `$facts['networking']['fqdn']` |
| `$hostname` | `$facts['networking']['hostname']` |
| `$domain` | `$facts['networking']['domain']` |
| `$ipaddress` | `$facts['networking']['ip']` |
| `$ipaddress6` | `$facts['networking']['ip6']` |
| `$macaddress` | `$facts['networking']['mac']` |
| `$memorysize` | `$facts['memory']['system']['total']` |
| `$memorysize_mb` | `$facts['memory']['system']['total_bytes']` / 1048576 |
| `$processorcount` | `$facts['processors']['count']` |
| `$architecture` | `$facts['os']['architecture']` |
| `$kernel` | `$facts['kernel']` |
| `$kernelversion` | `$facts['kernelversion']` |
| `$virtual` | `$facts['virtual']` |
| `$is_virtual` | `$facts['is_virtual']` |
| `$selinux` | `$facts['os']['selinux']['enabled']` |
| `$uptime_hours` | `$facts['system_uptime']['hours']` |

**Search and replace pattern:**

```bash
# Find legacy fact usage in your codebase
grep -rE '\$(osfamily|operatingsystem|fqdn|hostname|ipaddress|memorysize|processorcount)' modules/
```

**Tool:** Use [puppet-lint](https://rubygems.org/gems/puppet-lint) to automatically detect legacy fact usage:

```bash
gem install puppet-lint
puppet-lint manifests/
```

---

### 3. Hiera 3 Functions are Removed

The legacy Hiera 3 lookup functions have been **completely removed**. Use `lookup()` instead.

| ❌ Removed | ✅ Replacement |
|------------|----------------|
| `hiera('key')` | `lookup('key')` |
| `hiera('key', 'default')` | `lookup('key', String, 'first', 'default')` |
| `hiera_array('key')` | `lookup('key', Array, 'unique')` |
| `hiera_hash('key')` | `lookup('key', Hash, 'hash')` |
| `hiera_include('classes')` | `lookup('classes', Array[String], 'unique').include` |

**Example migration:**

```puppet
# Before (Puppet 7)
$packages = hiera_array('profile::base::packages')
$db_config = hiera_hash('myapp::database')
hiera_include('classes')

# After (Puppet 8 / OpenVox 8)
$packages = lookup('profile::base::packages', Array[String], 'unique')
$db_config = lookup('myapp::database', Hash, 'deep')
lookup('classes', Array[String], 'unique').include
```

> **Pro tip:** The automatic parameter lookup feature still works the same way — class parameters are automatically looked up from Hiera using the `classname::parameter` pattern. You don't need to change anything for that.

---

### 4. Ruby Version Updated

OpenVox 8 ships with **Ruby 3.2.x** (Puppet 7 used Ruby 2.7.x). This affects:

- Custom facts written in Ruby
- Custom functions
- Custom types and providers

**Common issues:**

- `URI.escape` is removed — use `CGI.escape` or `URI.encode_www_form_component`
- Keyword arguments must use explicit `**kwargs` syntax
- Some gems may need updates for Ruby 3.x compatibility

---

### 5. OpenSSL 3.0

OpenVox 8 uses **OpenSSL 3.0.x** instead of 1.1.1. This affects:

- SSL/TLS cipher suites
- Certificate handling
- Some older TLS 1.0/1.1 connections may fail

If you have custom integrations that use SSL, test them thoroughly.

---

## Migration Checklist

Use this checklist when migrating from Puppet 7 to OpenVox 8:

- [ ] **Audit legacy facts** — Search for `$osfamily`, `$operatingsystem`, `$fqdn`, etc.
- [ ] **Replace with structured facts** — Use `$facts['os']['family']`, etc.
- [ ] **Audit Hiera 3 functions** — Search for `hiera(`, `hiera_array(`, `hiera_hash(`
- [ ] **Replace with lookup()** — See conversion table above
- [ ] **Run puppet-lint** — `gem install puppet-lint && puppet-lint .`
- [ ] **Run puppet parser validate** — Check for undefined variable references
- [ ] **Test custom Ruby code** — Facts, functions, types, providers
- [ ] **Update module dependencies** — Ensure all Forge modules support Puppet 8
- [ ] **Test in a non-production environment first** — Always!

---

## Tools for Migration

### puppet-lint

The community linter for OpenVox/Puppet code. It detects:

- Legacy fact usage
- Deprecated Hiera 3 functions
- Style guide violations
- Common anti-patterns

```bash
# Install
gem install puppet-lint

# Run on a module
puppet-lint modules/mymodule/

# Run on entire codebase
puppet-lint .
```

### puppet parser validate

Built-in syntax validation — now catches undefined variables:

```bash
puppet parser validate manifests/site.pp
puppet parser validate modules/mymodule/manifests/*.pp
```

### PDK (Puppet Development Kit)

Validates modules against current standards:

```bash
pdk validate --parallel
pdk test unit --parallel
```

---

## Package Migration

> **Important:** You **cannot** install Puppet and OpenVox packages on the same host at the same time — they collide on `/etc/puppetlabs/` and `/opt/puppetlabs/`. Always **back up `/etc/puppetlabs/`** before swapping packages, especially on server nodes (the CA lives there, and losing it forces every agent to re-enroll).

### Upgrade Order (Multi-Host Sites)

When upgrading or migrating a fleet, do it in this order so central services stay ahead of the agents they serve:

1. **`openvox-server`** — upgrade the catalog compiler first
2. **`openvoxdb`** — upgrade the data warehouse next
3. **`openvoxdb-termini`** — on each server node so it can talk to the new OpenVoxDB
4. **`openvox-agent`** — finally, roll out to managed nodes

After each stage:

- Confirm the affected service is running
- Run a test agent execution (`puppet agent -t --noop`)
- Check certificate handling and OpenVoxDB connectivity where applicable
- Review the [OpenVox release notes](https://github.com/OpenVoxProject/openvox/releases) for the version you're moving to

If you're migrating from Puppet packages to OpenVox packages:

### RHEL / CentOS / Rocky / AlmaLinux

```bash
# Remove old Puppet repository
sudo yum remove puppet-release puppet7-release

# Install OpenVox repository
sudo rpm -Uvh https://yum.voxpupuli.org/openvox8-release-el-$(rpm -E %{rhel}).noarch.rpm

# Replace packages
sudo yum swap puppet-agent openvox-agent
sudo yum swap puppetserver openvox-server  # If running a server
```

### Debian / Ubuntu

```bash
# Remove old Puppet repository
sudo rm /etc/apt/sources.list.d/puppet*.list

# Install OpenVox repository
wget https://apt.voxpupuli.org/openvox8-release-$(lsb_release -cs).deb
sudo dpkg -i openvox8-release-$(lsb_release -cs).deb
sudo apt-get update

# Replace packages
sudo apt-get install openvox-agent  # Replaces puppet-agent
sudo apt-get install openvox-server # Replaces puppetserver (if needed)
```

### Configuration

Your existing configuration files work as-is:

- `/etc/puppetlabs/puppet/puppet.conf` — No changes needed
- `/etc/puppetlabs/code/` — Your environments, modules, and Hiera data work unchanged
- SSL certificates — Continue working (same CA)

---

## Getting Help

- **OpenVox Documentation** — You're reading it!
- **VoxPupuli Slack** — [voxpupuli.slack.com](https://voxpupuli.slack.com)
- **Puppet Community Slack** — [puppetcommunity.slack.com](https://puppetcommunity.slack.com)
- **GitHub Issues** — [github.com/openvoxproject](https://github.com/openvoxproject)

---

*Back to: [Getting Started →](README.md)*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
