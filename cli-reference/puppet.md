# puppet — The OpenVox CLI

> *The Swiss Army knife of infrastructure management.*

[← Back to CLI Reference](README.md)

---

The `puppet` command is the primary CLI for OpenVox. It uses a subcommand structure: `puppet <subcommand> [options]`.

## `puppet --help`

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

## `puppet --version`

```bash
$ sudo /opt/puppetlabs/puppet/bin/puppet --version
```

Real output from our sample infrastructure:

```
8.25.0
```

---

*See the individual subcommand pages for detailed help, options, and usage examples.*

[← Back to CLI Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>