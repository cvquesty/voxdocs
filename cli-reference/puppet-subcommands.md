# Additional puppet Subcommands

> *Less common, but still important.*

[← Back to CLI Reference](README.md)

---

These specialized subcommands cover operations you'll use less frequently but are good to know about.

## `puppet facts`

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

## `puppet catalog`

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

## `puppet epp`

```
$ puppet help epp

USAGE: puppet epp <action>

Interact directly with the EPP template parser/renderer.

ACTIONS:
  dump        Outputs a dump of the internal template parse tree for debugging
  render      Renders an epp template as text
  validate    Validate the syntax of one or more EPP templates.
```

## `puppet node`

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

## `puppet describe`

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

## `puppet filebucket`

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

## `puppet device`

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

## `puppet script`

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

## `puppet generate`

```
$ puppet help generate

USAGE: puppet generate <action>

Generates Puppet code from Ruby definitions.

ACTIONS:
  types    Generates Puppet code for custom types
```

> **Pro tip on `puppet generate types`:** This is often called automatically by r10k via the `--generate-types` flag. It creates `.pp` files from Ruby type definitions so PuppetServer can parse them without loading Ruby — a significant performance improvement.

[← Back to CLI Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>