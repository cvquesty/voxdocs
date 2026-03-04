# ⌨️ CLI Reference

> *Every binary, every subcommand, every flag. Because `--help` is never quite enough.*
>
> **All output on this page is real.** Captured from `openvox.example.com` running OpenVox 8.25.0 on RHEL 9.7.

---

## Overview

OpenVox ships with several command-line tools. This reference covers every binary you'll encounter, organized by component. Each entry includes the **complete, unedited help output** from a live system, plus practical usage examples.

**Binaries covered:**

| Binary | Version | What It Does |
|--------|---------|-------------|
| [`puppet`](#puppet) | 8.25.0 | The Swiss Army knife — agent, apply, resource, config, ssl, module, and more |
| [`facter`](#facter) | 5.4.0 | Cross-platform system fact discovery |
| [`puppetserver`](#puppetserver) | 8.12.1 | Server management, CA operations |
| [`puppet lookup`](#puppet-lookup) | (built-in) | Hiera data lookups from the CLI |
| [`r10k`](#r10k) | 5.0.2 | Code deployment from Git to environments |
| [`bolt`](#bolt-openbolt) | 5.3.0 | Agentless orchestration (OpenBolt) |

---

## puppet

The `puppet` command is the primary CLI for OpenVox. It uses a subcommand structure: `puppet <subcommand> [options]`.

### `puppet --help`

```
$ puppet --help

Usage: puppet <subcommand> [options] <action> [options]

Available subcommands:

  Common:
    agent             The puppet agent daemon provided by OpenVox
    apply             Apply Puppet manifests locally via OpenVox
    config            Interact with OpenVox's settings.
    help              Display OpenVox help.
    lookup            Interactive Hiera lookup for OpenVox
    module            Creates, installs and searches for modules on the Puppet Forge.
    resource          The OpenVox resource abstraction layer shell


  Specialized:
    catalog           Compile, save, view, and convert catalogs.
    describe          Display help about resource types available to OpenVox
    device            Manage remote network devices via OpenVox
    doc               Generate Puppet references for OpenVox
    epp               Interact directly with the EPP template parser/renderer.
    facts             Retrieve and store facts.
    filebucket        Store and retrieve files in an OpenVox filebucket
    generate          Generates Puppet code from Ruby definitions.
    node              View and manage node definitions.
    parser            Interact directly with the parser.
    plugin            Interact with the OpenVox plugin system.
    script            Run a puppet manifests as a script without compiling a catalog
    ssl               Manage SSL keys and certificates for OpenVox SSL clients

See 'puppet help <subcommand> <action>' for help on a specific subcommand action.
See 'puppet help <subcommand>' for help on a specific subcommand.
OpenVox v8.25.0
```

> **Notice the branding:** OpenVox 8.25.0 identifies itself as "OpenVox" throughout its help output, not "Puppet." The subcommands and behavior are identical — it's the same codebase with community governance.

### `puppet --version`

```
$ puppet --version
8.25.0
```

---

### `puppet agent`

The agent daemon — connects to the Primary Server, retrieves a catalog, and applies it. This is the command you'll type more than any other in your OpenVox career.

```
$ puppet help agent

puppet-agent(8) -- The puppet agent daemon provided by OpenVox
========

SYNOPSIS
--------
Retrieves the client configuration from the OpenVox server and applies it to
the local host.

This service may be run as a daemon, run periodically using cron (or something
similar), or run interactively for testing purposes.

USAGE
-----
puppet agent [--certname <NAME>] [-D|--daemonize|--no-daemonize]
  [-d|--debug] [--detailed-exitcodes] [--digest <DIGEST>] [--disable [MESSAGE]] [--enable]
  [--fingerprint] [-h|--help] [-l|--logdest syslog|eventlog|<ABS FILEPATH>|console]
  [--serverport <PORT>] [--noop] [-o|--onetime] [--sourceaddress <IP_ADDRESS>] [-t|--test]
  [-v|--verbose] [-V|--version] [-w|--waitforcert <SECONDS>]

DESCRIPTION
-----------
This is the main puppet client. Its job is to retrieve the local
machine's configuration from a remote server and apply it. In order to
successfully communicate with the remote server, the client must have a
certificate signed by a certificate authority that the server trusts;
the recommended method for this, at the moment, is to run a certificate
authority as part of the puppet server (which is the default). The
client will connect and request a signed certificate, and will continue
connecting until it receives one.

Once the client has a signed certificate, it will retrieve its
configuration and apply it.

OPTIONS
-------
Note that any Puppet setting that's valid in the configuration file is also a
valid long argument. For example, 'server' is a valid setting, so you can
specify '--server <servername>' as an argument. Boolean settings accept a '--no-'
prefix to turn off a behavior.

* --certname:        Set the certname (unique ID) of the client.
* --daemonize:       Send the process into the background (default).
* --no-daemonize:    Do not send the process into the background.
* --debug:           Enable full debugging.
* --detailed-exitcodes: Provide extra information via exit codes (see below).
* --digest:          Change the certificate fingerprint digest algorithm (default: SHA256).
* --disable [MSG]:   Disable puppet agent runs (optional reason message).
* --enable:          Re-enable puppet agent runs.
* --evaltrace:       Log each resource as it is evaluated.
* --fingerprint:     Display the current certificate fingerprint and exit.
* --help:            Print this help message.
* --job-id:          Attach a job ID to the catalog request and report (--onetime only).
* --logdest:         Where to send log messages (syslog, eventlog, console, or file path).
* --noop:            Dry-run mode — show what would change, change nothing.
* --onetime:         Run the configuration once and exit.
* --serverport:      Port to contact the OpenVox server on (default: 8140).
* --sourceaddress:   Set the source IP for outbound connections.
* --test:            Run in test mode (--onetime --verbose --no-daemonize
                     --no-usecacheonfailure --detailed-exitcodes --no-splay --show_diff).
* --trace:           Print stack traces on errors.
* --verbose:         Turn on verbose reporting.
* --version:         Print the puppet version number and exit.
* --waitforcert:     Seconds to wait for cert signing (default: 120, 0 to disable).
* --write_catalog_summary: Save resource/class lists to state directory after compilation.

SIGNALS
-------
  SIGHUP:   Restart the puppet agent daemon.
  SIGINT:   Shut down the puppet agent daemon.
  SIGTERM:  Shut down the puppet agent daemon.
  SIGUSR1:  Immediately retrieve and apply configurations.
  SIGUSR2:  Close and reopen log file descriptors (for logrotate).

COPYRIGHT
---------
Copyright (c) 2011 Puppet Inc.
Copyright (c) 2024 Vox Pupuli
Licensed under the Apache 2.0 License
```

**Common usage patterns:**

```bash
# Run once in test mode — the command you'll type 10,000 times
sudo puppet agent -t

# Dry run — see what WOULD change without changing anything
sudo puppet agent -t --noop

# Run against a specific server
sudo puppet agent -t --server=openvox.example.com

# Run in a specific environment (great for testing feature branches)
sudo puppet agent -t --environment=staging

# Disable the agent with a reason (shows up in reports)
sudo puppet agent --disable "Maintenance window - admin 2026-03-04"

# Re-enable the agent
sudo puppet agent --enable

# Check the certificate fingerprint
sudo puppet agent --fingerprint

# Run with full debug output (prepare for a LOT of text)
sudo puppet agent -t --debug

# Run with resource-level timing (find slow resources)
sudo puppet agent -t --evaltrace

# Only apply resources tagged with a specific class
sudo puppet agent -t --tags 'profile::webserver'
```

**Exit codes** (with `--detailed-exitcodes` or `--test`):

| Code | Meaning | CI/CD Treatment |
|------|---------|-----------------|
| `0` | No changes — system already in desired state | ✅ Success |
| `1` | An error occurred or run was blocked | ❌ Failure |
| `2` | Changes were applied successfully | ✅ Success (not an error!) |
| `4` | Some resources failed | ❌ Failure |
| `6` | Changes applied AND failures occurred | ❌ Failure |

> **⚠️ Classic gotcha:** Exit code **2** means "I made changes and they all worked." It is NOT an error! Many CI/CD systems treat any non-zero exit code as failure. See [Troubleshooting](../troubleshooting/README.md) for the fix.

---

### `puppet apply`

Compile and apply a manifest locally — no server needed. Perfect for standalone work, testing, and learning.

```
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

**Common usage patterns:**

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

---

### `puppet resource`

Inspect and manage resources directly from the command line. This is like having X-ray vision for your system — you can see the current state of *any* resource type.

```
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

**Common usage patterns:**

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

**Live example** from `openvox.example.com`:

```
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

---

### `puppet config`

Read and modify Puppet configuration settings. Think of it as the `git config` of the Puppet world.

```
$ puppet help config

USAGE: puppet config <action> [--section SECTION_NAME]

This subcommand can inspect and modify settings from OpenVox's
'puppet.conf' configuration file.

OPTIONS:
  --render-as FORMAT             - The rendering format to use.
  --verbose                      - Whether to log verbosely.
  --debug                        - Whether to log debug information.
  --section SECTION_NAME         - The section of the configuration file to
                                   interact with.

ACTIONS:
  delete    Delete an OpenVox setting.
  print     Examine OpenVox's current settings.
  set       Set OpenVox's settings.
```

**Common usage patterns:**

```bash
# Print ALL settings (there are hundreds)
puppet config print all

# Print a specific setting
puppet config print server
puppet config print certname
puppet config print runinterval
puppet config print modulepath
puppet config print environmentpath

# Set the server hostname
sudo puppet config set server openvox.example.com --section agent

# Set the run interval to 1 hour
sudo puppet config set runinterval 3600 --section agent

# Set the environment
sudo puppet config set environment production --section agent

# Delete a setting (revert to default)
sudo puppet config delete server --section agent
```

---

### `puppet ssl`

Manage SSL certificates and keys for agent-server authentication. This is how you handle the PKI (Public Key Infrastructure) that underpins all OpenVox communication.

```
$ puppet help ssl

puppet-ssl(8) -- Manage SSL keys and certificates for OpenVox SSL clients
========

SYNOPSIS
--------
Manage SSL keys and certificates for clients needing
to communicate with an OpenVox infrastructure.

USAGE
-----
puppet ssl <action> [-h|--help] [-v|--verbose] [-d|--debug] [--localca] [--target CERTNAME]

OPTIONS
-------
* --help:            Print this help message.
* --verbose:         Print extra information.
* --debug:           Enable full debugging.
* --localca:         Also clean the local CA certificate and CRL.
* --target CERTNAME: Clean the specified device certificate instead of this host's certificate.

ACTIONS
-------
* bootstrap:        Perform all steps to request and download a client certificate.
                    If autosigning is disabled, puppet will wait every `waitforcert`
                    seconds for its certificate to be signed. Specify 0 to never wait.
* submit_request:   Generate a CSR and submit it to the CA.
* generate_request: Generate a CSR (but don't submit it).
* download_cert:    Download a signed certificate for this host.
* verify:           Verify the private key and certificate match, and the cert is trusted.
* clean:            Remove the private key and certificate files for this host.
                    With --localca, also remove the local CA cert and CRL bundle.
                    With --target, clean a specific device certificate.
* show:             Print the full-text version of this host's certificate.

COPYRIGHT
---------
Copyright (c) 2011 Puppet Inc.
Copyright (c) 2024 Vox Pupuli
Licensed under the Apache 2.0 License
```

**Common usage patterns:**

```bash
# Bootstrap SSL (generate key + CSR, request signing)
sudo puppet ssl bootstrap

# Show current certificate info
sudo puppet ssl show

# Verify the certificate chain
sudo puppet ssl verify

# Clean local SSL data (for re-registration)
sudo puppet ssl clean

# Submit a new CSR
sudo puppet ssl submit_request
```

---

### `puppet module`

Install, search, and manage Puppet modules from the Forge. Modules are how you leverage the community's work — thousands of pre-built modules are available for everything from Apache to ZFS.

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

**Common usage patterns:**

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

---

### `puppet parser`

Validate and interact with Puppet manifests. This is your syntax-checking safety net — run it before every commit.

```
$ puppet help parser

USAGE: puppet parser <action>

Interact directly with the parser.

OPTIONS:
  --render-as FORMAT             - The rendering format to use.
  --verbose                      - Whether to log verbosely.
  --debug                        - Whether to log debug information.

ACTIONS:
  dump        Outputs a dump of the internal parse tree for debugging
  validate    Validate the syntax of one or more Puppet manifests.
```

**Common usage patterns:**

```bash
# Validate a manifest (check for syntax errors)
puppet parser validate manifest.pp

# Validate all .pp files in a directory
find . -name '*.pp' -exec puppet parser validate {} +

# Dump the AST (Abstract Syntax Tree)
puppet parser dump manifest.pp
```

---

### `puppet describe`

Get documentation about resource types and their parameters.

```bash
# List all available resource types
puppet describe --list

# Get detailed info about a resource type
puppet describe file
puppet describe package
puppet describe service
puppet describe user
puppet describe exec

# Short description only
puppet describe --short file
```

---

### `puppet facts`

Query and manage system facts.

```bash
# Show all facts
puppet facts

# Show facts in YAML format
puppet facts --render-as yaml

# Show a specific fact
puppet facts show os.name
puppet facts show networking.ip
```

---

### `puppet lookup`

Perform Hiera data lookups from the command line. This is the modern replacement for the deprecated `hiera` CLI, and it's *the* debugging tool for Hiera. If you're wondering "where is this value coming from?", `puppet lookup --explain` is your new best friend.

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

**Common usage patterns:**

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

**Example output with `--explain`:**

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

---

## facter

Facter is the cross-platform fact-gathering tool. It discovers everything about your system — hardware, OS, networking, virtualization, cloud metadata — and makes it available to Puppet as variables. Think of it as `uname`, `lscpu`, `ip addr`, and `dmidecode` all rolled into one structured output.

### `facter --help`

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

### `facter --version`

```
$ facter --version
5.4.0
```

**Common usage patterns:**

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

**Live example** from `openvox.example.com`:

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

---

## puppetserver

The PuppetServer binary manages the JVM-based server process and the built-in Certificate Authority. This runs on the Primary Server only.

### `puppetserver --help`

```
$ puppetserver --help

usage: puppetserver ([--help] | [--version]) <command> [<args>]

The most commonly used puppetserver commands are:
   ca
   foreground
   gem
   irb
   prune
   reload
   ruby
   start
   stop

See 'puppetserver <command> -h' for more information on a specific command.
```

### `puppetserver version`

```
$ puppetserver version
puppetserver version: 8.12.1
```

### `puppetserver ca`

The built-in Certificate Authority management tool. This is how you manage the PKI infrastructure that secures all agent↔server communication.

```
$ puppetserver ca --help

Usage: puppetserver ca <action> [options]

Manage the Private Key Infrastructure for
Puppet Server's built-in Certificate Authority

Available Actions:

  Certificate Actions (requires a running Puppet Server):

    clean       Revoke cert(s) and remove related files from CA
    generate    Generate a new certificate signed by the CA
    list        List certificates and CSRs
    revoke      Revoke certificate(s)
    sign        Sign certificate request(s)

  Administrative Actions (requires Puppet Server to be stopped):

    delete      Delete signed certificate(s) from disk
    import      Import an external CA chain and generate server PKI
    setup       Setup a self-signed CA chain for Puppet Server
    enable      Setup infrastructure CRL based on a node inventory.
    migrate     Migrate the existing CA directory
    prune       Prune the local CRL on disk to remove certificate entries

General Options:
        --help       Display this general help output
        --version    Display the version
        --verbose    Display low-level information
```

**Common usage patterns:**

```bash
# List all certificate signing requests (pending + signed)
sudo puppetserver ca list --all

# List only pending (unsigned) requests
sudo puppetserver ca list

# Sign a specific certificate request
sudo puppetserver ca sign --certname agent1.example.com

# Sign ALL pending requests
sudo puppetserver ca sign --all

# Revoke a certificate (compromised or decommissioned node)
sudo puppetserver ca revoke --certname old-server.example.com

# Clean a certificate (remove from CA entirely)
sudo puppetserver ca clean --certname old-server.example.com

# Generate a certificate for a specific node
sudo puppetserver ca generate --certname new-service.example.com
```

**Live example** from `openvox.example.com`:

```
$ sudo puppetserver ca list --all

Signed Certificates:
    openvox.example.com       (SHA256)  F9:70:1B:30:19:46:10:5D:7A:19:41:94:8D:40:92:34:...
        alt names: ["DNS:puppet", "DNS:openvox.example.com"]
        authorization extensions: [pp_cli_auth: true]
    agent1.example.com        (SHA256)  94:2C:B9:EA:C4:16:98:0A:52:D2:71:BA:3E:BC:76:56:...
        alt names: ["DNS:agent1.example.com"]
    agent2.example.com        (SHA256)  17:26:8C:66:4D:B0:43:F4:96:FE:D0:D4:72:FB:C3:37:...
        alt names: ["DNS:agent2.example.com"]
```

> **Pro tip:** Notice the server cert has `alt names` including `puppet` — this is the `dns_alt_names` setting. The `pp_cli_auth: true` extension means this cert can be used for CLI-based CA operations.

---

## r10k

r10k (pronounced "are-ten-kay") deploys Puppet environments from Git branches and installs modules from the Forge. The name is a reference to the Mandalorian R10K assassin droids — because this tool is killer at deployment. (No, seriously, that's the actual origin.)

### `r10k help`

```
$ r10k help

NAME
    r10k - Killer robot powered Puppet environment deployment

USAGE
    r10k <subcommand> [options]

DESCRIPTION
    r10k is a suite of commands to help deploy and manage puppet code for
    complex environments.

COMMANDS
    deploy         Puppet dynamic environment deployment
    help           show help
    puppetfile     Perform operations on a Puppetfile
    version        Print the version of r10k

OPTIONS
    -c --config=<value>         Specify a global configuration file
       --color                  Enable colored log messages
    -h --help                   Show help for this command
    -t --trace                  Display stack traces on application crash
    -v --verbose[=<value>]      Set log verbosity. Valid values: fatal,
                                error, warn, notice, info, debug, debug1,
                                debug2
```

### `r10k version`

```
$ r10k version
r10k 5.0.2
```

### `r10k deploy`

```
$ r10k help deploy

NAME
    deploy - Puppet dynamic environment deployment

USAGE
    r10k deploy <subcommand>

DESCRIPTION
    `r10k deploy` implements the Git branch to Puppet environment workflow.

SUBCOMMANDS
    display         Display environments and modules in the deployment
    environment     Deploy environments and their dependent modules
    module          Deploy modules in all environments

OPTIONS
       --cachedir=<value>              Specify a cachedir, overriding the value in config
       --exclude-spec[=<value>]        Exclude the module's spec dir for deployment
       --generate-types                Run `puppet generate types` after updating an environment
       --github-app-id=<value>         Github App id. Only valid with rugged provider
       --github-app-key=<value>        Github App private key. Only valid with rugged provider
       --no-force                      Prevent the overwriting of local module modifications
       --oauth-token=<value>           Path to OAuth token for cloning (rugged provider only)
       --private-key=<value>           Path to SSH key for cloning (rugged provider only)
       --puppet-conf=<value>           Path to puppet.conf
       --puppet-path=<value>           Path to puppet executable
```

### `r10k puppetfile`

```
$ r10k help puppetfile

NAME
    puppetfile - Perform operations on a Puppetfile

USAGE
    r10k puppetfile <subcommand>

DESCRIPTION
    `r10k puppetfile` provides an implementation of the librarian-puppet
    style Puppetfile.

SUBCOMMANDS
    check       Try and load the Puppetfile to verify the syntax is correct.
    install     Install all modules from a Puppetfile
    purge       Purge unmanaged modules from a Puppetfile managed directory
```

**Common usage patterns:**

```bash
# Deploy ALL environments (the most common command)
sudo r10k deploy environment --verbose

# Deploy a specific environment
sudo r10k deploy environment production --verbose

# Deploy environments AND update modules from Puppetfile
sudo r10k deploy environment --modules --verbose

# Deploy a single module across all environments
sudo r10k deploy module stdlib --verbose

# Show what would be deployed (display current state)
r10k deploy display

# Install modules from Puppetfile (in current directory)
r10k puppetfile install --verbose

# Check Puppetfile syntax
r10k puppetfile check

# Purge unmanaged modules
r10k puppetfile purge
```

**Example r10k configuration** (`/etc/puppetlabs/r10k/r10k.yaml`):

```yaml
---
# Sources define Git repos that map branches to environments
sources:
  control:
    remote: 'git@github.com:myorg/control-repo.git'
    basedir: '/etc/puppetlabs/code/environments'
    prefix: false

# Where to cache Git repos
cachedir: '/opt/puppetlabs/puppet/cache/r10k'
```

---

## bolt (OpenBolt)

Bolt — now **OpenBolt** in the OpenVox ecosystem — is an agentless orchestration tool. It connects to targets via SSH (or WinRM for Windows) and runs commands, scripts, tasks, and plans without requiring a Puppet agent. Think of it as Ansible's opinionated cousin who insists on doing things the Puppet way.

### `bolt --help`

```
$ bolt --help

Name
    OpenBolt

Usage
    bolt <subcommand> [action] [options]

Description
    OpenBolt is an orchestration tool that automates the manual work it takes to
    maintain your infrastructure.

Subcommands
    apply             Apply Puppet manifest code
    command           Run a command remotely
    file              Copy files between the controller and targets
    group             Show the list of groups in the inventory
    guide             View guides for Bolt concepts and features
    inventory         Show the list of targets an action would run on
    module            Manage Bolt project modules
    lookup            Look up a value with Hiera
    plan              Convert, create, show, and run Bolt plans
    plugin            Show available plugins
    policy            Apply, create, and show policies
    project           Create and migrate Bolt projects
    script            Upload a local script and run it remotely
    secret            Create encryption keys and encrypt and decrypt values
    task              Show and run Bolt tasks

Guides
    For a list of guides on Bolt's concepts and features, run 'bolt guide'.
    Find Bolt's documentation at https://bolt.guide.

Global options
    -h, --help                       Display help.
        --version                    Display the version.
        --log-level LEVEL            Set the log level for the console. Available options are
                                     trace, debug, info, warn, error, fatal.
        --clear-cache                Clear plugin, plan, and task caches before executing.
```

### `bolt --version`

```
$ bolt --version
5.3.0
```

> **Note:** The community fork is called **OpenBolt** and identifies itself as such in the `--help` output. It includes additional subcommands like `policy` and `plugin` not present in older Puppet Bolt versions.

**Common usage patterns:**

```bash
# Run a command on remote targets
bolt command run 'uptime' --targets webservers

# Run a command on a single host
bolt command run 'df -h' --targets agent1.example.com

# Upload a file
bolt file upload /local/path/file.conf /remote/path/file.conf --targets all

# Download a file
bolt file download /var/log/messages ./logs/ --targets agent1.example.com

# Run a script
bolt script run ./deploy.sh --targets webservers

# Run a Puppet task
bolt task run package action=install name=httpd --targets webservers

# Run a Bolt plan
bolt plan run myapp::deploy version=2.0 --targets webservers

# Apply a Puppet manifest (agentless!)
bolt apply manifest.pp --targets agent1.example.com

# Apply inline Puppet code
bolt apply -e 'package { "vim": ensure => installed }' --targets all

# Show inventory
bolt inventory show --targets all

# Lookup Hiera data
bolt lookup myapp::db_password --targets dbserver.example.com

# Use PuppetDB for target discovery
bolt command run 'hostname' --query 'nodes[certname] { facts.os.name = "Rocky" }'
```

**Connection options:**

```bash
# SSH with specific user and key
bolt command run 'id' --targets host.example.com \
  --user deploy --private-key ~/.ssh/id_ed25519

# WinRM for Windows targets
bolt command run 'Get-Service' --targets winhost.example.com \
  --transport winrm --user Administrator --password

# Using an inventory file
bolt command run 'uptime' --targets all -i inventory.yaml

# Limit concurrency
bolt command run 'apt update' --targets all --concurrency 10

# Output as JSON
bolt command run 'hostname -f' --targets all --format json
```

---

## Additional puppet Subcommands

These specialized subcommands cover less common but still important operations.

### `puppet facts`

```
$ puppet help facts

USAGE: puppet facts <action> [--terminus _TERMINUS]

This subcommand manages facts, which are collections of normalized system
information used by OpenVox. It can read facts directly from the local system
(with the default `facter` terminus).

ACTIONS:
  find      Retrieve a node's facts.
  info      Print the default terminus class for this face.
  save      API only: create or overwrite an object.
  show      Retrieve current node's facts.
  upload    Upload local facts to the puppet master.

TERMINI: facter, json, memory, network_device, puppetdb, puppetdb_apply, rest, store_configs, yaml
```

### `puppet catalog`

```
$ puppet help catalog

USAGE: puppet catalog <action> [--terminus _TERMINUS]

This subcommand deals with catalogs, which are compiled per-node artifacts
generated from a set of Puppet manifests.

ACTIONS:
  apply       Find and apply a catalog.
  compile     Compile a catalog.
  diff        Compare catalogs from different puppet versions.
  download    Download this node's catalog from the puppet master server.
  find        Retrieve the catalog for the node from which the command is run.
  save        API only: create or overwrite an object.
  seed        Generate a series of catalogs
  select      Retrieve a catalog and filter it for resources of a given type.

TERMINI: compiler, json, msgpack, puppetdb, rest, store_configs, yaml
```

### `puppet epp`

```
$ puppet help epp

USAGE: puppet epp <action>

Interact directly with the EPP template parser/renderer.

ACTIONS:
  dump        Outputs a dump of the internal template parse tree for debugging
  render      Renders an epp template as text
  validate    Validate the syntax of one or more EPP templates.
```

### `puppet node`

```
$ puppet help node

USAGE: puppet node <action> [--terminus _TERMINUS]

This subcommand interacts with node objects, which are used by OpenVox to
build a catalog.

ACTIONS:
  clean         Clean up signed certs, cached facts, node objects, and reports
  deactivate    Deactivate a set of nodes in PuppetDB
  find          Retrieve a node object.
  status        Fetch the current status for a set of nodes in PuppetDB

TERMINI: exec, json, memory, msgpack, plain, puppetdb, rest, store_configs, yaml
```

### `puppet describe`

```
$ puppet help describe

puppet-describe(8) -- Display help about resource types available to OpenVox
========

USAGE
-----
puppet describe [-h|--help] [-s|--short] [-p|--providers] [-l|--list] [-m|--meta]

OPTIONS
-------
* --help:       Print this help text
* --providers:  Describe providers in detail for each type
* --list:       List all types
* --meta:       List all metaparameters
* --short:      List only parameters without detail
```

### `puppet filebucket`

```
$ puppet help filebucket

puppet-filebucket(8) -- Store and retrieve files in an OpenVox filebucket
========

USAGE
-----
puppet filebucket <mode> [options] <file> <file> ...

Modes: backup, get, restore, diff, list

OPTIONS
-------
* --bucket:   Specify a local filebucket path.
* --local:    Use the local filebucket.
* --remote:   Use a remote filebucket.
* --server:   The server to use for file storage.
* --fromdate: (list only) Select bucket files from this date.
* --todate:   (list only) Select bucket files until this date.
```

### `puppet device`

```
$ puppet help device

puppet-device(8) -- Manage remote network devices via OpenVox
========

USAGE
-----
puppet device [-h|--help] [-v|--verbose] [-d|--debug]
  [-l|--logdest syslog|<file>|console] [--detailed-exitcodes]
  [--deviceconfig <file>] [-w|--waitforcert <seconds>]
  [-a|--apply <file>] [-f|--facts] [-r|--resource <type> [name]]
  [-t|--target <device>] [--user=<user>]

DESCRIPTION
-----------
Devices require a proxy OpenVox agent to request certificates, collect facts,
retrieve and apply catalogs, and store reports. Configured in device.conf.
```

### `puppet script`

```
$ puppet help script

puppet-script(8) -- Run a puppet manifests as a script without compiling a catalog
========

USAGE
-----
puppet script [-h|--help] [-V|--version] [-d|--debug] [-v|--verbose]
  [-e|--execute] [-l|--logdest syslog|eventlog|<FILE>|console] [--noop] <file>

DESCRIPTION
-----------
This is a standalone puppet script runner tool; use it to run puppet code
without compiling a catalog. When provided with a modulepath, puppet script
can load functions, types, tasks and plans from modules.
```

### `puppet generate`

```
$ puppet help generate

USAGE: puppet generate <action>

Generates Puppet code from Ruby definitions.

ACTIONS:
  types    Generates Puppet code for custom types
```

> **Pro tip on `puppet generate types`:** This is often called automatically by r10k via the `--generate-types` flag. It creates `.pp` files from Ruby type definitions so PuppetServer can parse them without loading Ruby — a significant performance improvement.

---

## puppet query (PuppetDB)

Query PuppetDB using PQL (Puppet Query Language). PQL is like SQL, but for your infrastructure — and once you learn it, you'll wonder how you ever lived without it.

```bash
# Find all nodes
puppet query 'nodes {}'

# Find nodes by OS
puppet query 'nodes[certname] { facts.os.name = "RedHat" }'

# Find nodes with a specific package
puppet query 'resources[certname] { type = "Package" and title = "httpd" }'

# Find facts for a node
puppet query 'facts { certname = "agent1.example.com" }'

# Find nodes that changed on last run
puppet query 'nodes[certname] { latest_report_status = "changed" }'

# Find nodes that haven't checked in for 2 hours
puppet query 'nodes[certname] { report_timestamp < "2 hours ago" }'

# Count nodes by OS family
puppet query 'facts[value, count()] { name = "os.family" group by value }'

# Find all nodes with a specific class applied
puppet query 'resources[certname] { type = "Class" and title = "Profile::Base" }'

# Export PuppetDB data
puppet db export backup.tgz

# Import PuppetDB data
puppet db import backup.tgz
```

---

## Utility Commands Cheat Sheet

Quick reference for day-to-day operations:

```bash
# ─── Agent Operations ───
sudo puppet agent -t                    # Test run
sudo puppet agent -t --noop             # Dry run
sudo puppet agent --disable "reason"    # Lock agent
sudo puppet agent --enable              # Unlock agent

# ─── Local Apply ───
sudo puppet apply manifest.pp           # Apply a manifest
sudo puppet apply -e 'code'             # Apply inline code
puppet parser validate manifest.pp      # Check syntax

# ─── Inspection ───
puppet resource user                    # List all users
puppet resource package httpd           # Check package state
puppet resource service sshd            # Check service state
puppet describe file                    # Docs for file type

# ─── Configuration ───
puppet config print server              # Show server setting
puppet config print all                 # Show all settings
sudo puppet config set server host.com  # Set server

# ─── Facts ───
facter                                  # All facts
facter os.name                          # Specific fact
facter --json                           # JSON output

# ─── SSL ───
sudo puppet ssl show                    # Show certificate
sudo puppet ssl verify                  # Verify cert chain
sudo puppet ssl clean                   # Remove local certs

# ─── Modules ───
puppet module list                      # List installed
sudo puppet module install author-mod   # Install from Forge
puppet module search keyword            # Search Forge

# ─── Server / CA ───
sudo puppetserver ca list               # Pending CSRs
sudo puppetserver ca list --all         # All certs
sudo puppetserver ca sign --certname X  # Sign a CSR
sudo puppetserver ca revoke --certname X # Revoke a cert
sudo puppetserver ca clean --certname X # Remove a cert

# ─── Hiera Lookup ───
puppet lookup myclass::param            # Lookup a key
puppet lookup key --explain             # Show hierarchy
puppet lookup key --merge deep          # Deep merge

# ─── Code Deployment ───
sudo r10k deploy environment --verbose  # Deploy all envs
sudo r10k deploy environment prod -v    # Deploy one env

# ─── Orchestration (Bolt) ───
bolt command run 'cmd' --targets host   # Run command
bolt task run task --targets host       # Run task
bolt plan run plan --targets host       # Run plan
```

---

*Next up: [Configuration Reference →](../configuration/README.md)*
