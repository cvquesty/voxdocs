# puppet lookup

> *The debugging tool for Hiera. "Where is this value coming from?" — answered.*

[← Back to CLI Reference](README.md)

---

Perform Hiera data lookups from the command line. This is the modern replacement for the deprecated `hiera` CLI.

```
$ puppet help lookup

puppet-lookup(8) -- Interactive Hiera lookup for OpenVox
========

SYNOPSIS
--------
Does Hiera lookups from the command line.

Since this command needs access to your Hiera data, make sure to run it on a
node that has a copy of that data. This usually means logging into an OpenVox
server node and running 'puppet lookup' with sudo.

The most common version of this command is:

'puppet lookup <KEY> --node <NAME> --environment <ENV> --explain'

USAGE
-----
puppet lookup [--help] [--type <TYPESTRING>] [--merge first|unique|hash|deep]
  [--knock-out-prefix <PREFIX-STRING>] [--sort-merged-arrays]
  [--merge-hash-arrays] [--explain] [--environment ENV]
  [--default <VALUE>] [--node <NODE-NAME>] [--facts <FILE>]
  [--compile] [--render-as s|json|yaml|binary|msgpack] <keys>

OPTIONS
-------
* --merge <STRATEGY>:    Merge strategy: first, unique, hash, deep (default: first).
* --explain:             Show the lookup path and where the value came from.
* --explain-options:     Show how lookup_options affect this lookup.
* --environment <ENV>:   The environment to look up in (default: production).
* --default <VALUE>:     Default value if key is not found.
* --node <NODE>:         Simulate lookup for a specific node (uses PuppetDB facts if available).
* --facts <FILE>:        Override facts with a JSON or YAML file.
* --render-as <FORMAT>:  Output format: s (plain text), json, yaml, binary, msgpack.
* --compile:             Perform a full catalog compilation (slower but more accurate).
* --type <TYPE>:         Assert the return type (e.g., String, Integer, Hash).
* --knock-out-prefix:    Prefix string to remove items during deep merge.
* --sort-merged-arrays:  Sort all merged arrays (with deep merge).
* --merge-hash-arrays:   Deep-merge hashes within arrays by position.

COPYRIGHT
---------
Copyright (c) 2015 Puppet Inc.
Copyright (c) 2024 Vox Pupuli
Licensed under the Apache 2.0 License
```

## Common Usage Patterns

```bash
# Simple lookup
puppet lookup myclass::my_parameter

# Lookup with explanation (shows the hierarchy path)
puppet lookup myclass::db_host --explain

# Lookup in a specific environment
puppet lookup myclass::port --environment staging

# Lookup with merge strategy
puppet lookup myclass::servers --merge unique

# Deep merge a hash
puppet lookup myclass::config --merge deep

# Lookup for a specific node
puppet lookup myclass::role --node webserver1.example.com

# Lookup with default value
puppet lookup myclass::optional_param --default "fallback_value"

# Output as JSON
puppet lookup myclass::all_settings --render-as json
```

## Example Output with `--explain`

```
$ puppet lookup ntp::servers --explain
Searching for "ntp::servers"
  Global Data Provider (hiera configuration version 5)
    Using configuration "/etc/puppetlabs/puppet/hiera.yaml"
    Hierarchy entry "Per-node data"
      Path "/etc/puppetlabs/code/environments/production/data/nodes/openvox.example.com.yaml"
        Original path: "nodes/%{facts.networking.fqdn}.yaml"
        No such key: "ntp::servers"
    Hierarchy entry "OS family"
      Path "/etc/puppetlabs/code/environments/production/data/os/RedHat.yaml"
        Original path: "os/%{facts.os.family}.yaml"
        Found key: "ntp::servers" value: ["0.rhel.pool.ntp.org", "1.rhel.pool.ntp.org"]
```

[← Back to CLI Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>