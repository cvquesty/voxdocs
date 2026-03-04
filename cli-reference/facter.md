# facter

> *`uname`, `lscpu`, `ip addr`, and `dmidecode` all rolled into one.*

[← Back to CLI Reference](README.md)

---

Facter is the cross-platform fact-gathering tool. It discovers everything about your system — hardware, OS, networking, virtualization, cloud metadata — and makes it available to Puppet as variables.

## `facter --help`

```
$ facter --help

Usage
=====

  facter [options] [query] [query] [...]

Options
=======
           [--color]                      Enable color output.
           [--no-color]                   Disable color output.
        -c [--config]                     The location of the config file.
           [--custom-dir]                 A directory to use for custom facts.
        -d [--debug]                      Enable debug output.
           [--external-dir]               A directory to use for external facts.
           [--hocon]                      Output in Hocon format.
        -j [--json]                       Output in JSON format.
        -l [--log-level]                  Set logging level. Supported levels are: none, trace,
                                          debug, info, warn, error, and fatal.
           [--no-block]                   Disable fact blocking.
           [--no-cache]                   Disable loading and refreshing facts from the cache
           [--no-custom-facts]            Disable custom facts.
           [--no-external-facts]          Disable external facts.
           [--no-ruby]                    Disable loading Ruby, facts requiring Ruby, and custom facts.
           [--trace]                      Enable backtraces for custom facts.
           [--verbose]                    Enable verbose (info) output.
           [--show-legacy]                Show legacy facts when querying all facts.
        -y [--yaml]                       Output in YAML format.
           [--strict]                     Enable more aggressive error reporting.
        -t [--timing]                     Show how much time it took to resolve each fact
           [--sequential]                 Resolve facts sequentially
           [--http-debug]                 Whether to write HTTP request and responses to stderr.
        -p [--puppet]                     Load the Puppet libraries, thus allowing Facter to load
                                          Puppet-specific facts.
        -v [--version]                    Print the version
           [--list-block-groups]          List block groups
           [--list-cache-groups]          List cache groups
        -h [--help]                       Help for all arguments
```

## `facter --version`

```
$ facter --version
5.4.0
```

## Common Usage Patterns

```bash
# Show ALL facts (long output!)
facter

# Show all facts as JSON
facter --json

# Query specific facts (dot notation)
facter os.name
facter os.release.full
facter os.family
facter networking.ip
facter networking.fqdn
facter networking.interfaces
facter processors.count
facter memory.system.total
facter memory.system.available
facter disks
facter virtual
facter identity.user

# Multiple facts at once
facter os.name os.release.full networking.ip

# Include Puppet-specific facts
facter -p

# Show timing for each fact (useful for debugging slow fact collection)
facter -t os

# Output as YAML
facter -y os
```

## Live Example

From `openvox.example.com`:

```
$ facter os
{
  architecture => "x86_64",
  distro => {
    codename => "Plow",
    description => "Red Hat Enterprise Linux release 9.7 (Plow)",
    id => "RedHatEnterprise",
    release => {
      full => "9.7",
      major => "9",
      minor => "7"
    }
  },
  family => "RedHat",
  hardware => "x86_64",
  name => "RedHat",
  release => {
    full => "9.7",
    major => "9",
    minor => "7"
  },
  selinux => {
    config_mode => "enforcing",
    config_policy => "targeted",
    current_mode => "enforcing",
    enabled => true,
    enforced => true,
    policy_version => "33"
  }
}
```

```
$ facter networking.fqdn networking.ip memory.system.total processors.count virtual
memory.system.total => 15.15 GiB
networking.fqdn => openvox.example.com
networking.ip => 192.168.1.100
processors.count => 4
virtual => physical
```

> **Notice** how Facter 5.x returns structured data with nested hashes. You can query specific sub-keys using dot notation like `os.release.major`. This is exactly how you'll access these values in your Puppet manifests: `$facts['os']['release']['major']`.

[← Back to CLI Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>