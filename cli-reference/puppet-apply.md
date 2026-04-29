# puppet apply

> *No server required — just you, a manifest, and `sudo`.*

[← Back to CLI Reference](README.md)

---

Compile and apply a manifest locally — no server needed. Perfect for standalone work, testing, and learning.

```console
$ puppet help apply

puppet-apply(8) -- Apply Puppet manifests locally via OpenVox
========

SYNOPSIS
--------
Applies a standalone Puppet manifest to the local system.

USAGE
-----
puppet apply [-h|--help] [-V|--version] [-d|--debug] [-v|--verbose]
  [-e|--execute] [--detailed-exitcodes] [-L|--loadclasses]
  [-l|--logdest syslog|eventlog|<ABS FILEPATH>|console] [--noop]
  [--catalog <catalog>] [--write-catalog-summary] <file>

DESCRIPTION
-----------
This is the standalone puppet execution tool; use it to apply
individual manifests.

When provided with a modulepath, via command line or config file, puppet
apply can effectively mimic the catalog that would be served by OpenVox
server with access to the same modules, although there are some subtle
differences. When combined with scheduling and an automated system for
pushing manifests, this can be used to implement a serverless site.

OPTIONS
-------
* --debug:                  Enable full debugging.
* --detailed-exitcodes:     Provide extra information via exit codes (same as puppet agent).
* --execute:                Execute a specific piece of Puppet code.
* --help:                   Print this help message.
* --loadclasses:            Load any stored classes from the agent's classes.txt cache.
* --logdest:                Where to send log messages (syslog, eventlog, console, or file path).
                            Supports JSON (.json) and JSON Lines (.jsonl) output.
* --noop:                   Dry-run mode — show what would change, change nothing.
* --catalog:                Apply a pre-compiled JSON catalog instead of compiling from a manifest.
* --test:                   Run in test mode (--verbose, --detailed-exitcodes, --show_diff).
* --verbose:                Print extra information.
* --write-catalog-summary:  Save resource/class lists to disk after compilation.

COPYRIGHT
---------
Copyright (c) 2011 Puppet Inc.
Copyright (c) 2024 Vox Pupuli
Licensed under the Apache 2.0 License
```

## Common Usage Patterns

```bash
# Apply a manifest file
sudo puppet apply site.pp

# Execute Puppet code inline (great for quick tests!)
sudo puppet apply -e 'package { "vim": ensure => installed }'

# Dry run a manifest — see what WOULD change
sudo puppet apply --noop webserver.pp

# Apply with debug output (very verbose)
sudo puppet apply -d manifest.pp

# Apply with a custom modulepath
sudo puppet apply --modulepath=/opt/modules manifest.pp

# Apply with structured JSON logging
sudo puppet apply --logdest /var/log/puppet-apply.jsonl manifest.pp
```

> **Pro tip:** `puppet apply` is the best way to learn the Puppet language. Write a `.pp` file, apply it, see what happens. Rinse, repeat. No server required.

[← Back to CLI Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
