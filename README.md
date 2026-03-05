# 📘 OpenVox Documentation

> **The unofficial, community-written documentation for the OpenVox project.**
>
> *Because great software deserves great docs — and a few good jokes along the way.*

---

## What Is OpenVox?

[OpenVox](https://github.com/openvoxproject) is a **community-driven, open-source configuration management platform** — a continuation of Puppet's open-source heritage, maintained by the [Vox Pupuli](https://voxpupuli.org/) community. If you've ever used Puppet, you already know OpenVox. It's the same powerful declarative language, the same agent/server architecture, and the same ecosystem of thousands of modules on the [Puppet Forge](https://forge.puppet.com/). The difference? OpenVox is **100% open source**, forever, with no corporate strings attached.

Think of OpenVox as Puppet's cooler, community-owned sibling who actually shows up to the family reunion.

### How Does OpenVox Relate to Puppet?

| | **OpenVox** | **Puppet (Perforce)** |
|---|---|---|
| **License** | Apache 2.0 (fully open) | Mixed (open source + proprietary PE) |
| **Governance** | Community (Vox Pupuli) | Corporate (Perforce) |
| **Compatibility** | Drop-in replacement for Puppet 8.x | N/A |
| **Modules** | All Puppet Forge modules work | Puppet Forge |
| **Packages** | yum/apt.voxpupuli.org | puppet.com |
| **History** | Forked from open-source Puppet | Original product by Puppet Labs |

> **Key takeaway:** If you're running open-source Puppet today, OpenVox is a seamless replacement. Your manifests, modules, Hiera data, and `puppet.conf` don't need to change. Swap the packages and you're running OpenVox.

## Current Shipping Versions

These versions are currently shipping from the Vox Pupuli repositories, verified against a live OpenVox infrastructure on **RHEL 9.7**:

| Component | Version | Binary Path | Notes |
|-----------|---------|-------------|-------|
| **openvox-agent** | `8.25.0` | `/opt/puppetlabs/puppet/bin/puppet` | Ruby-based, all platforms |
| **openvox-server** | `8.12.1` | `/opt/puppetlabs/bin/puppetserver` | JRuby + Jetty, FIPS capable |
| **Facter** | `5.4.0` | `/opt/puppetlabs/puppet/bin/facter` | C++/Ruby fact discovery |
| **Hiera** | 5 (integrated) | Built into puppet | Hierarchical data lookup |
| **OpenBolt** | `5.3.0` | `/usr/local/bin/bolt` | Agentless orchestration |
| **r10k** | `5.0.2` | `/opt/puppetlabs/puppet/bin/r10k` | Git-to-environment deployer |
| **PuppetDB** | `8.x` | systemd service | PostgreSQL-backed data warehouse |

> **📌 Note:** All CLI output in this documentation is **real output** captured from `openvox.example.com` — a live three-node fleet running RHEL 9.7 with 4 CPUs, 15 GiB RAM, and SELinux in enforcing mode. Nothing here is fabricated.

## 📂 Documentation Map

This documentation is organized into focused guides. Read them in order for a learning path, or jump directly to what you need:

| # | Guide | What You'll Learn |
|---|-------|-------------------|
| 1 | [🚀 **Getting Started**](getting-started/README.md) | Installation, first steps, your first manifest |
| 2 | [🏗️ **Architecture & Concepts**](architecture/README.md) | How all the pieces fit together |
| 3 | [📝 **The Puppet Language**](language/README.md) | Resources, classes, defined types, and the DSL |
| 4 | [⌨️ **CLI Reference**](cli-reference/README.md) | Every binary, every flag — with real output |
| 5 | [⚙️ **Configuration Reference**](configuration/README.md) | `puppet.conf`, `hiera.yaml`, and all the knobs |
| 6 | [🖥️ **Server Administration**](server-admin/README.md) | PuppetServer, PuppetDB, the CA, and backups |
| 7 | [📚 **Hiera Deep-Dive**](hiera/README.md) | Hierarchical data, merge strategies, and eyaml |
| 8 | [📦 **Module Development**](module-development/README.md) | Writing, testing, and publishing modules |
| 9 | [🎯 **Orchestration**](orchestration/README.md) | OpenBolt, r10k, tasks, plans, and code deployment |
| 10 | [🔧 **Troubleshooting & FAQ**](troubleshooting/README.md) | When things go sideways (and they will) |
| 11 | [🤝 **Community & Contributing**](community/README.md) | How to get involved |

## Who Is This For?

- 🖥️ **System administrators** managing fleets of servers
- ⚙️ **DevOps engineers** building infrastructure-as-code pipelines
- 🎓 **Students and learners** exploring configuration management
- 🔄 **Puppet veterans** migrating to OpenVox
- 🤯 **Anyone** who's ever thought "there has to be a better way to manage 500 servers"

## A Note on Copyright & Originality

This documentation is an **original work**, written specifically for the OpenVox project by community contributors. Our sources:

- ✅ The [Puppet docs-archive](https://github.com/puppetlabs/docs-archive) — legacy Puppet 5.x documentation, which is **unencumbered by copyright** and freely available for anyone to use
- ✅ The official [OpenVox project](https://github.com/openvoxproject) repositories, source code, and community resources
- ✅ Hands-on experience with real OpenVox infrastructure (verified against `openvox.example.com`)
- ✅ The Puppet language specification itself (which is open-source and part of the OpenVox codebase)

> **⚠️ Important:** This is **NOT** a copy, derivative, or adaptation of Perforce's official Puppet documentation at [puppet.com](https://puppet.com). While we cover similar topics (because it's the same technology), every explanation, example, and turn of phrase is originally written. We describe concepts in our own words, using our own examples from our own infrastructure.

## Package Repositories

OpenVox packages are available from the Vox Pupuli repositories:

| Platform | Repository URL | Supported Releases |
|----------|---------------|-------------------|
| **RHEL / CentOS / Rocky / AlmaLinux** | [yum.voxpupuli.org](https://yum.voxpupuli.org) | EL 8, 9, 10 |
| **Fedora** | [yum.voxpupuli.org](https://yum.voxpupuli.org) | 42+ |
| **Debian** | [apt.voxpupuli.org](https://apt.voxpupuli.org) | 11 (Bullseye), 12 (Bookworm) |
| **Ubuntu** | [apt.voxpupuli.org](https://apt.voxpupuli.org) | 22.04 (Jammy), 24.04 (Noble) |

## Key Links

| | Resource | URL |
|---|----------|-----|
| 🦊 | **OpenVox Project** | [github.com/openvoxproject](https://github.com/openvoxproject) |
| 🐾 | **Vox Pupuli** | [voxpupuli.org](https://voxpupuli.org/) |
| 📦 | **Puppet Forge** | [forge.puppet.com](https://forge.puppet.com/) — all modules work with OpenVox! |
| 📚 | **Puppet docs-archive** | [github.com/puppetlabs/docs-archive](https://github.com/puppetlabs/docs-archive) |
| 💬 | **Community Slack** | [voxpupuli.slack.com](https://short.voxpupu.li/puppetcommunity_slack_signup/) |
| 🛠️ | **OpenVox GUI** | [github.com/cvquesty/openvox-gui](https://github.com/cvquesty/openvox-gui) — web management UI |
| 🔍 | **OpenVox Lint** | [rubygems.org/gems/openvox-lint](https://rubygems.org/gems/openvox-lint) — Puppet manifest linter |

---

## License

This documentation is released under the [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/) (CC BY-SA 4.0).

You are free to share and adapt this material, provided you give appropriate credit and distribute your contributions under the same license.

---

*Built with ☕, sudo access, and mild frustration by the community. Last updated: March 2026.*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
