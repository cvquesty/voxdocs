# puppet module

> *Thousands of pre-built modules — from Apache to ZFS.*

[← Back to CLI Reference](README.md)

---

Install, search, and manage Puppet modules from the Forge.

```
$ puppet help module

USAGE: puppet module <action> [--environment production ] [--modulepath  ]

This subcommand can find, install, and manage modules from the Puppet Forge,
a repository of user-contributed Puppet code. It can also generate empty
modules, and prepare locally developed modules for release on the Forge.

OPTIONS:
  --render-as FORMAT             - The rendering format to use.
  --verbose                      - Whether to log verbosely.
  --debug                        - Whether to log debug information.
  --environment production       - The environment in which Puppet is running.
  --modulepath                   - The search path for modules.

ACTIONS:
  changes      Show modified files of an installed module.
  install      Install a module from the Puppet Forge or a release archive.
  list         List installed modules
  uninstall    Uninstall a puppet module.
  upgrade      Upgrade a puppet module.
```

## Common Usage Patterns

```bash
# List installed modules
puppet module list

# Install a module from the Forge
sudo puppet module install puppetlabs-apache

# Install a specific version
sudo puppet module install puppetlabs-apache --version 12.1.0

# Search for modules
puppet module search ntp

# Upgrade a module
sudo puppet module upgrade puppetlabs-stdlib

# Uninstall a module
sudo puppet module uninstall puppetlabs-motd

# Generate a new module skeleton
puppet module generate myorg-newmodule
```

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>