# puppet resource

> *X-ray vision for your system — see the current state of any resource type.*

[← Back to CLI Reference](README.md)

---

Inspect and manage resources directly from the command line.

```console
$ puppet help resource

puppet-resource(8) -- The OpenVox resource abstraction layer shell
========

SYNOPSIS
--------
Uses the OpenVox RAL to directly interact with the system.

USAGE
-----
puppet resource [-h|--help] [-d|--debug] [-v|--verbose] [-e|--edit]
  [-p|--param <parameter>] [-t|--types] [-y|--to_yaml] <type>
  [<name>] [<attribute>=<value> ...]

DESCRIPTION
-----------
This command provides simple facilities for converting current system
state into Puppet code, along with some ability to modify the current
state using Puppet's RAL.

By default, you must at least provide a type to list, in which case
puppet resource will tell you everything it knows about all resources of
that type. You can optionally specify an instance name, and puppet
resource will only describe that single instance.

If given a type, a name, and a series of <attribute>=<value> pairs,
puppet resource will modify the state of the specified resource.

OPTIONS
-------
* --debug:     Enable full debugging.
* --edit:      Write results to a file and open in $EDITOR.
* --help:      Print this help message.
* --param:     Add more parameters to be outputted from queries.
* --types:     List all available resource types.
* --to_yaml:   Output in YAML format (useful with Hiera and create_resources).
* --verbose:   Print extra information.
* --fail:      Return exit code 1 if the resource could not be modified.

COPYRIGHT
---------
Copyright (c) 2011 Puppet Inc.
Copyright (c) 2024 Vox Pupuli
Licensed under the Apache 2.0 License
```

## Common Usage Patterns

```bash
# List all users on the system
sudo puppet resource user

# Show details for a specific user
sudo puppet resource user root

# Show a specific package's state
sudo puppet resource package httpd

# Show a file's current state
sudo puppet resource file /etc/hostname

# Show a service's current state
sudo puppet resource service sshd

# CREATE a user (yes, you can modify the system this way!)
sudo puppet resource user testuser ensure=present shell=/bin/bash managehome=true

# Remove a user
sudo puppet resource user testuser ensure=absent

# List all available resource types
puppet resource --types

# Output in YAML (great for feeding into Hiera)
sudo puppet resource user root --to_yaml
```

## Live Example

From `openvox.example.com`:

```console
$ sudo puppet resource user root
user { 'root':
  ensure             => 'present',
  comment            => 'root',
  gid                => 0,
  home               => '/root',
  password           => '$6$OKJdrlECHHo/Vkpr$IRkVgKwwSi2...',
  password_max_age   => 99999,
  password_min_age   => 0,
  password_warn_days => 7,
  provider           => 'useradd',
  shell              => '/bin/bash',
  uid                => 0,
}
```

> **Pro tip:** `puppet resource` is incredibly useful for discovering the current state of a system before writing manifests. Want to know what packages are installed? `puppet resource package`. Want to see all services? `puppet resource service`. It's like `grep` for infrastructure.

[← Back to CLI Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
