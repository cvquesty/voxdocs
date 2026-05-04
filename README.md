# 📘 OpenVox Documentation

> **The unofficial, community-written documentation for the OpenVox project.**
>
> *Because great software deserves great docs — and a few good jokes along the way.*

---

## What Is OpenVox?

[OpenVox](https://github.com/openvoxproject) is a **community-driven, open-source configuration management platform** — a continuation of Puppet's open-source heritage, maintained by the [Vox Pupuli](https://voxpupuli.org/) community. If you've ever used Puppet, you already know OpenVox. It's the same powerful declarative language, the same agent/server architecture, and the same ecosystem of thousands of modules on the [Puppet Forge](https://forge.puppet.com/). The difference? OpenVox is **100% open source**, forever, with no corporate strings attached.

Think of OpenVox as Puppet's cooler, community-owned sibling who actually shows up to the family reunion.

### A Brief History

When Perforce discontinued public distribution of open-source Puppet in late 2024, the community needed a path forward. **Overlook InfraTech** stepped in first with community packaging so existing Puppet users wouldn't be stranded. The project was then adopted under [Vox Pupuli](https://voxpupuli.org/) stewardship and renamed **OpenVox**. A **Puppet Standards Steering Committee** now guides language and feature evolution, ensuring continued compatibility across the broader Puppet ecosystem.

The result: a fully open, community-governed continuation of the platform, with the same DSL, the same Forge, and the same operational patterns you already know.

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
| **openvox-agent** | `8.26.2` | `/opt/puppetlabs/puppet/bin/puppet` | Ruby-based, all platforms |
| **openvox-server** | `8.12.1` | `/opt/puppetlabs/bin/puppetserver` | JRuby + Jetty, FIPS capable |
| **OpenFact** (was Facter) | `5.6.0` | `/opt/puppetlabs/puppet/bin/facter` | C++/Ruby fact discovery; binary still `facter` |
| **Hiera** | 5 (integrated) | Built into puppet | Hierarchical data lookup |
| **OpenBolt** (was Bolt) | `5.4.0` | `/usr/local/bin/bolt` | Agentless orchestration; package now `openbolt` |
| **r10k** | `5.0.2` | `/opt/puppetlabs/puppet/bin/r10k` | Git-to-environment deployer |
| **OpenVoxDB** (was PuppetDB) | `8.8.1` | systemd service | PostgreSQL-backed data warehouse; packages `openvoxdb`, `openvoxdb-termini` |

The `openvox-agent` 8.26.2 package bundles **Ruby 3.2.11** and **OpenSSL 3.0.20**.

### Supported Platforms

OpenVox publishes official `openvox-agent` packages for these platforms (per the [official OpenVox docs](https://github.com/OpenVoxProject/openvox-docs)):

| Platform | Supported Versions |
|----------|-------------------|
| **RHEL / CentOS / Rocky / AlmaLinux** | 7, 8, 9, 10 |
| **SLES** | 15, 16 |
| **Debian** | 10 (Buster), 11 (Bullseye), 12 (Bookworm), 13 |
| **Ubuntu** | 18.04, 20.04, 22.04, 24.04, 25.04, 26.04 |
| **Fedora** | 36, 40, 41, 42, 43 |
| **Windows** | Server 2008R2, 2012R2, 2016; Desktop 10 Enterprise |
| **macOS** | 10.12+, Intel and Apple Silicon |

Ruby 3.2.x is the only tested interpreter. OpenVox also runs (without official packages) on Gentoo, Arch, Solaris 10+, AIX 6.1+, FreeBSD 4.7+, and HP-UX.

> **📌 Note:** All CLI output in this documentation is **real output** captured from a live OpenVox server running RHEL 9.x with SELinux in enforcing mode. Nothing here is fabricated. Hardware details (CPU count, RAM) are representative of typical production setups.

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

This documentation is an **original work**, written specifically for the OpenVox project by community contributors. It is a **community companion** to the official [OpenVox documentation](https://github.com/OpenVoxProject/openvox-docs), not a fork or copy of it. Our sources:

- ✅ The [official OpenVox documentation](https://github.com/OpenVoxProject/openvox-docs) (CC BY-SA 3.0, copyright Puppet, Inc.) -- referenced for version numbers, platform support tables, component naming, and project-level facts such as release dates and known issues
- ✅ The [Puppet docs-archive](https://github.com/puppetlabs/docs-archive) -- legacy Puppet 5.x documentation, which is **unencumbered by copyright** and freely available for anyone to use
- ✅ The official [OpenVox project](https://github.com/openvoxproject) repositories, source code, and community resources
- ✅ Hands-on experience with real OpenVox infrastructure (verified against `openvox.example.com`)
- ✅ The Puppet language specification itself (which is open-source and part of the OpenVox codebase)

> **⚠️ Important:** This is **NOT** a copy, derivative, or adaptation of Perforce's official Puppet documentation at [puppet.com](https://puppet.com), nor is it a fork of the official OpenVox documentation project. While we cover similar topics (because it's the same technology) and reference the official docs for factual accuracy, every explanation, example, and turn of phrase is originally written. We describe concepts in our own words, using our own examples from our own infrastructure.

## Package Repositories

OpenVox packages are available from the Vox Pupuli repositories:

| Platform | Repository URL | Supported Releases |
|----------|---------------|-------------------|
| **RHEL / CentOS / Rocky / AlmaLinux** | [yum.voxpupuli.org](https://yum.voxpupuli.org) | EL 8, 9, 10 |
| **Fedora** | [yum.voxpupuli.org](https://yum.voxpupuli.org) | 42+ |
| **Debian** | [apt.voxpupuli.org](https://apt.voxpupuli.org) | 11 (Bullseye), 12 (Bookworm) |
| **Ubuntu** | [apt.voxpupuli.org](https://apt.voxpupuli.org) | 22.04 (Jammy), 24.04 (Noble) |
| **Windows** | [downloads.voxpupuli.org/windows](https://downloads.voxpupuli.org/windows) | MSI installers |
| **macOS** | [downloads.voxpupuli.org/mac](https://downloads.voxpupuli.org/mac) | PKG installers (Intel + Apple Silicon) |

For the canonical, always-up-to-date install instructions across every platform,
see the official [Installing OpenVox](https://voxpupuli.org/openvox/install/) guide.

## Key Links

| | Resource | URL |
|---|----------|-----|
| 🦊 | **OpenVox Project** | [github.com/openvoxproject](https://github.com/openvoxproject) |
| 📥 | **Installing OpenVox** | [voxpupuli.org/openvox/install](https://voxpupuli.org/openvox/install/) |
| 🛟 | **OpenVox Support** | [voxpupuli.org/openvox/support](https://voxpupuli.org/openvox/support/) |
| 📚 | **Official OpenVox Docs** | [github.com/OpenVoxProject/openvox-docs](https://github.com/OpenVoxProject/openvox-docs) |
| 🐾 | **Vox Pupuli** | [voxpupuli.org](https://voxpupuli.org/) |
| 📦 | **Puppet Forge** | [forge.puppet.com](https://forge.puppet.com/) — all modules work with OpenVox! |
| 📚 | **Puppet docs-archive** | [github.com/puppetlabs/docs-archive](https://github.com/puppetlabs/docs-archive) |
| 💬 | **Community Slack** | [puppetcommunity.slack.com](https://puppetcommunity.slack.com/) |
| 🛠️ | **OpenVox GUI** | [github.com/cvquesty/openvox-gui](https://github.com/cvquesty/openvox-gui) — web management UI |
| 🔍 | **OpenVox Lint** | [rubygems.org/gems/openvox-lint](https://rubygems.org/gems/openvox-lint) — Puppet manifest linter |

---

## License

This documentation is released under the [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/) (CC BY-SA 4.0).

You are free to share and adapt this material, provided you give appropriate credit and distribute your contributions under the same license.

### Attribution

This project references the following works:

- **[OpenVox Documentation](https://github.com/OpenVoxProject/openvox-docs)** -- Copyright (c) Puppet, Inc. Licensed under the [Creative Commons Attribution-ShareAlike 3.0 International License](https://creativecommons.org/licenses/by-sa/3.0/) (CC BY-SA 3.0). Version numbers, platform support tables, component naming conventions, release dates, and known issues are sourced from this project.
- **[Puppet docs-archive](https://github.com/puppetlabs/docs-archive)** -- Legacy Puppet 5.x documentation, unencumbered by copyright and freely available for public use. Referenced for historical context and foundational concepts.

All original text, examples, and explanations in this repository are the work of the VoxDocs community contributors and are licensed under CC BY-SA 4.0 as stated above.

---

*Built with coffee, sudo access, and mild frustration by the community. Last updated: May 2026.*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
