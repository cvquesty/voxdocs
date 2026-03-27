# ⌨️ CLI Reference

> *Every binary, every subcommand, every flag. Because `--help` is never quite enough.*
>
> **All output in these pages is real.** Captured from a live OpenVox server running 8.25.0 on RHEL 9.7.

---

## Command Reference

Each command has its own page with the complete `--help` output and practical usage examples.

### Core Commands

| # | Command | Version | Description |
|---|---------|---------|-------------|
| 1 | [**puppet**](puppet.md) | 8.25.0 | The main OpenVox CLI — overview and subcommand list |
| 2 | [**puppet agent**](puppet-agent.md) | | Agent daemon — connects to server, applies catalogs |
| 3 | [**puppet apply**](puppet-apply.md) | | Apply manifests locally (no server needed) |
| 4 | [**puppet resource**](puppet-resource.md) | | Inspect and manage system resources directly |
| 5 | [**puppet config**](puppet-config.md) | | Read and modify `puppet.conf` settings |
| 6 | [**puppet ssl**](puppet-ssl.md) | | Manage SSL certificates and keys |
| 7 | [**puppet module**](puppet-module.md) | | Install, search, and manage Forge modules |
| 8 | [**puppet parser**](puppet-parser.md) | | Validate manifest syntax |
| 9 | [**puppet lookup**](puppet-lookup.md) | | Hiera data lookups from the CLI |

### Server & Infrastructure

| # | Command | Version | Description |
|---|---------|---------|-------------|
| 10 | [**facter**](facter.md) | 5.4.0 | Cross-platform system fact discovery |
| 11 | [**puppetserver**](puppetserver.md) | 8.12.1 | Server management and Certificate Authority |
| 12 | [**r10k**](r10k.md) | 5.0.2 | Code deployment from Git to environments |
| 13 | [**bolt** (OpenBolt)](bolt.md) | 5.3.0 | Agentless orchestration |

### Additional References

| # | Page | Description |
|---|------|-------------|
| 14 | [**Additional puppet Subcommands**](puppet-subcommands.md) | facts, catalog, epp, node, describe, filebucket, device, script, generate |
| 15 | [**puppet query (PQL)**](puppet-query.md) | PuppetDB queries using Puppet Query Language |
| 16 | [**Cheat Sheet**](cheatsheet.md) | Quick-reference card for day-to-day operations |

---

*Next up: [Configuration Reference →](../configuration/README.md)*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
