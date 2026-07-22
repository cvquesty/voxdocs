# ⌨️ CLI Reference

> *Every binary, every subcommand, every flag. Because `--help` is never quite enough.*
>
> **All output in these pages is real.** Captured from a live OpenVox lab on RHEL 9.7 running the package set
> `openvox-agent-8.26.2`, `openvox-server-8.12.1`, `openvoxdb-8.12.1`, `openvoxdb-termini-8.12.1`, `openbolt-5.4.0`.
> Note: `puppet --version` reports `8.26.1` because of cosmetic bug
> [OpenVoxProject/openvox#415](https://github.com/OpenVoxProject/openvox/issues/415); the binary is genuinely 8.26.2.

---

## Command Reference

Each command has its own page with the complete `--help` output and practical usage examples.

### Core Commands

| # | Command | Version | Description |
|---|---------|---------|-------------|
| 1 | [**puppet**](puppet.md) | Latest 8.28.1 · lab 8.26.2 (binary reports 8.26.1) | The main OpenVox CLI — overview and subcommand list |
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
| 10 | [**facter**](facter.md) | Latest 5.7.0 · lab 5.6.0 | Cross-platform system fact discovery (OpenFact) |
| 11 | [**puppetserver**](puppetserver.md) | Latest 8.15.0 · lab 8.12.1 | Server management and Certificate Authority |
| 12 | [**r10k**](r10k.md) | 5.0.3 via OpenBolt · lab 5.0.2 | Code deployment from Git; **not** bundled in openvox-server |
| 13 | [**bolt** (OpenBolt)](bolt.md) | Latest 5.6.0 · lab 5.4.0 | Agentless orchestration; package `openbolt` |
| —  | **OpenVoxDB** (was PuppetDB) | Latest 8.15.0 · lab 8.12.1 | systemd service `puppetdb`; query via `puppet query` (row 15) |

### Additional References

| # | Page | Description |
|---|------|-------------|
| 14 | [**Additional puppet Subcommands**](puppet-subcommands.md) | facts, catalog, epp, node, describe, filebucket, device, script, generate |
| 15 | [**puppet query (PQL)**](puppet-query.md) | PuppetDB queries using Puppet Query Language |
| 16 | [**Cheat Sheet**](cheatsheet.md) | Quick-reference card for day-to-day operations |

---

*Next up: [Configuration Reference →](../configuration/README.md)*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
