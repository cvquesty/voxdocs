# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.1.1] - 2026-07-22

### Security / Privacy
- Redact private lab hostnames from public docs: `openvox.questy.org` → **a live lab**,
  agents → `*.example.com`, lab RFC1918 IPs → documentation examples
- Weekly mirror script sanitizes every sync so residual private FQDNs never publish

## [2.1.0] - 2026-07-22

### Changed
- **Production (voxdocs.questy.org)** now mirrors the official OpenVox docs site
  at [docs.openvoxproject.org](https://docs.openvoxproject.org/) (VitePress
  published HTML), instead of only the community Docsify Markdown set

### Added
- `SYNC.md` — how weekly automation works on server.questy.org
- Weekly systemd timer: **Saturday 02:00 America/New_York**
  (`voxdocs-sync.timer` → `voxdocs-sync-openvox.sh`)

## [2.0.8] - 2026-07-22

### Added
- **Docsify** documentation shell for interactive Markdown rendering on
  [voxdocs.questy.org](https://voxdocs.questy.org):
  - `index.html` — SPA shell with search, dark/light toggle, code copy,
    pagination, syntax highlighting (bash/ruby/puppet/yaml/json)
  - `_sidebar.md` — full guide navigation (11 sections)
  - `_navbar.md` — top nav + links to VoxForge and OpenVox
- Vox branding (VoxPupuli Blue `#0D6EFD`, Orange `#EC8622`)

### Changed
- Site hosting moved to `server.questy.org` (Apache static + Docsify)
- Root serves rendered docs hub instead of raw Markdown files

## [2.0.7] - 2026-05-23

### Daily currency pass — versions, releases, and lab verification

OpenVoxProject ships updates frequently. This pass ensures every "current shipping" claim, version table, and descriptive reference matches the absolute latest GitHub release tags + the exact running state of the primary lab infrastructure.

- Fresh SSH capture from `a live lab` (RHEL 9.7, 2026-05-23): confirmed package set `openvox-agent-8.26.2`, `openvox-server-8.12.1`, `openvoxdb-8.12.1`+termini, `openbolt-5.4.0`; all services (`puppetserver`, `puppetdb`, `puppet`) active; binaries and paths unchanged.
- Upstream GitHub latest (as of tool run): `openvox-server` 8.13.0, `openvoxdb` 8.13.0, `openbolt` 5.5.0 (agent still 8.26.2 in this cycle).
- Updated "Current Shipping Versions" table in README.md to show **Latest Shipping** vs **Lab Verified (2026-05-23)** with explicit footnote and capture date.
- Synced AGENTS.md "Current Verified Versions" section with the same latest-vs-lab distinction.
- Updated version references in cli-reference/README.md table, cli-reference/bolt.md, and orchestration/README.md to reflect 8.13.0 / 5.5.0 latest while preserving exact captured output (8.12.1 / 5.4.0) for all `--version` and `--help` blocks.
- Confirmed `puppetserver` and `puppetdb` remain the correct systemd unit names even with `openvox-*` packages (our prior service-name fixes in getting-started/server-admin/troubleshooting remain accurate).
- No new core packages, paths, or breaking changes detected in the capture.

All documentation output blocks continue to be 100% real lab captures. "Current" claims now explicitly separate "what the repos ship today" from "what this repo's examples were captured against."

## [2.0.5] - 2026-05-04

### Alignment pass against the official OpenVox documentation project

Reviewed and synthesized content from the official OpenVox documentation
([github.com/OpenVoxProject/openvox-docs](https://github.com/OpenVoxProject/openvox-docs))
into the community docs.

#### Version updates
- README.md: OpenVoxDB version updated from "8.x" to 8.8.1 (per openvox-docs release notes)
- README.md: Added Ruby 3.2.11 / OpenSSL 3.0.20 component details for openvox-agent 8.26.2
- AGENTS.md: Added OpenVoxDB 8.8.1 to verified versions list

#### Platform support
- README.md: Added full Supported Platforms table sourced from openvox-docs system requirements (RHEL 7-10, SLES 15-16, Debian 10-13, Ubuntu 18.04-26.04, Fedora 36-43, Windows, macOS, plus unofficial platforms)

#### PDK alternatives
- module-development/README.md: Replaced bare PDK section with "PDK and Open-Source Alternatives" noting PDK 3.4.0 was the last OSS version; documented VoxBox (Vox Pupuli container image for CI/testing) and jig (Go-based PDK reimplementation)

#### Copyright and licensing
- README.md: Updated "A Note on Copyright & Originality" to reference openvox-docs as a source (CC BY-SA 3.0, copyright Puppet, Inc.) and clarify voxdocs is a community companion, not a fork
- README.md: Added Attribution subsection to License section with proper credit to openvox-docs and Puppet docs-archive
- AGENTS.md: Updated Copyright Policy to permit referencing openvox-docs for factual data (version numbers, platform support, component names, release dates, known issues)

## [2.0.6] - 2026-05-23

### Fixed — Service and package consistency with current OpenVox packaging

- **getting-started/README.md**: Corrected all `systemctl` references for the server from the incorrect `openvox-server` unit name to the actual `puppetserver` service name (the `openvox-server` package installs the `puppetserver` systemd unit for compatibility; confirmed via live lab and official install guide).
- **server-admin/README.md**: Updated the OpenVoxDB installation example to recommend the current packages `openvoxdb openvoxdb-termini` (was still showing legacy `puppetdb puppetdb-termini`); expanded the section header and installation text with branding and compatibility notes.
- **server-admin/README.md**: Rewrote the top-level overview to accurately describe the current package set (`openvox-server`, `openvoxdb`/`openvoxdb-termini`) while noting that service names (`puppetserver`, `puppetdb`) and paths remain unchanged.
- **troubleshooting/README.md**: Updated the PuppetDB troubleshooting section header and added a compatibility note explaining that `openvoxdb` is the package but `puppetdb` remains the service/unit name.
- **community/README.md**: Clarified that voxdocs is referenced by / included in the official OpenVox documentation project at OpenVoxProject/openvox-docs.

All changes verified against live lab infrastructure (openvox-agent 8.26.2 / server 8.12.1 / openvoxdb 8.12.1 / openbolt 5.4.0 on RHEL 9.7) and official Vox Pupuli installation documentation.

## [2.0.0] - 2026-04-27

### Major release: OpenVox project alignment

VoxDocs 2.0 brings the community documentation into alignment with the
official OpenVox documentation project ([github.com/OpenVoxProject/openvox-docs](https://github.com/OpenVoxProject/openvox-docs))
and the platform's current state on RHEL 9.7.

This is a **major version bump** because the conceptual framing of the
project has shifted: the OpenVox team has rebranded several core
components (Facter -> OpenFact, PuppetDB -> OpenVoxDB, Bolt -> OpenBolt),
the official origin story now credits Overlook InfraTech and the Puppet
Standards Steering Committee, and the documented package set has expanded.
Existing readers will encounter new terminology and a restructured
introduction. No documentation files were removed; readers can continue
using v1.x as a snapshot reference if needed.

This release rolls up two passes of alignment work.

### Wave 2 (2026-04-27): lab re-capture, OpenBolt 5.4.0, PostgreSQL recommendation

#### Lab re-capture (against the live lab, sanitized for publication)

- Verified live package set: `openvox-agent-8.26.2`, `openvox-server-8.12.1`, `openvoxdb-8.12.1`, `openvoxdb-termini-8.12.1`, `openbolt-5.4.0` on RHEL 9.7.
- Confirmed the 8.26.2 cosmetic version-string bug from OpenVoxProject/openvox#415 in the live lab: `puppet --version` reports `8.26.1` even though `rpm -q openvox-agent` reports 8.26.2. The version-string bug we documented in Wave 1 is real and reproducible.
- cli-reference/puppet.md: refreshed `puppet --help` tail line to `OpenVox v8.26.1`; updated `--version` block to `8.26.1` and added an inline note explaining the cosmetic bug + how to verify the real package version.
- cli-reference/README.md: rewrote the "All output is real" banner to enumerate the actual package set and explicitly call out the version-string bug. Updated puppet row to `8.26.2 (binary reports 8.26.1)`.
- getting-started/README.md: changed expected `puppet --version` output from `8.26.2` to `8.26.1` to match what users actually see; added rpm/dpkg verification commands.

#### OpenBolt version bump (5.3.0 -> 5.4.0)

- README.md (root) versions table, AGENTS.md, cli-reference/README.md, cli-reference/bolt.md, orchestration/README.md: all OpenBolt references bumped from 5.3.0 to 5.4.0.

#### PostgreSQL recommendation

- server-admin/README.md: changed "PuppetDB requires PostgreSQL 11+" to "OpenVoxDB requires PostgreSQL 11+, recommends PostgreSQL 14+" with PGDG repo links, matching the official OpenVoxDB 8 guidance.

#### vardir resolution

- The official OpenVox docs page `dirs_vardir.markdown` says agent vardir is `/var/opt/puppetlabs/puppet/cache`. Verified with the OpenVox docs maintainers: the correct value is `/opt/puppetlabs/puppet/cache` (matches what voxdocs has and what the live lab uses). Upstream PR pending against OpenVoxProject/openvox-docs. **No change needed in voxdocs.**

### Wave 1 (2026-04-21): alignment pass against the official OpenVox docs

#### Version bumps

- README.md / AGENTS.md / cli-reference: openvox-agent 8.25.0 -> 8.26.2 (per OpenVox release notes, 2026-04-18)
- README.md / AGENTS.md / cli-reference: Facter 5.4.0 -> OpenFact 5.6.0 (per OpenFact release notes, 2026-04-09)
- README.md footer: "Last updated" date moved to April 2026

#### OpenVox project rebranding

- AGENTS.md / architecture/README.md / cli-reference/facter.md: noted the official rename of Facter -> OpenFact, PuppetDB -> OpenVoxDB, Bolt -> OpenBolt. Binary names and config files (facter, puppetdb, bolt, facter.conf) are unchanged; only project/package names changed.
- architecture/README.md: added OpenFact, OpenVoxDB, and OpenBolt entries to the glossary.
- cli-reference/README.md: added OpenVoxDB row to the Server & Infrastructure table; noted openbolt package name on the bolt row.
- getting-started/README.md: added a "Full OpenVox Package Set" subsection covering openvox-agent, openvox-server, openvoxdb, openvoxdb-termini, openbolt.
- orchestration/README.md: documented the new openbolt package name alongside the legacy puppet-bolt name.

#### Origin story precision

- README.md: rewrote the project history paragraph to credit Overlook InfraTech for stepping in with community packaging when Perforce discontinued public distribution of open-source Puppet in late 2024, before Vox Pupuli adopted the project. Added mention of the Puppet Standards Steering Committee.
- community/README.md: expanded the acknowledgments list to call out Overlook InfraTech's role and the Standards Steering Committee.

#### Repos and downloads

- README.md: added Windows (downloads.voxpupuli.org/windows) and macOS (downloads.voxpupuli.org/mac) rows to the Package Repositories table; linked the official Installing OpenVox guide.
- README.md: added Installing OpenVox, OpenVox Support, and Official OpenVox Docs entries to the Key Links table.
- getting-started/README.md: added a "Windows and macOS Agents" subsection pointing at the download URLs.

#### Migration page

- getting-started/migration.md: added a "Two migration paths" callout summarizing the official upgrade routes (Puppet 7 -> OpenVox 7 -> OpenVox 8, or Puppet 7 -> Puppet 8 -> OpenVox 8).
- getting-started/migration.md: added an "Upgrade Order" subsection (server -> openvoxdb -> openvoxdb-termini -> agent) and a callout warning that Puppet and OpenVox cannot coexist on the same host.

#### Troubleshooting

- troubleshooting/README.md: added FAQ entry for the known cosmetic bug in OpenVox 8.26.2 where `puppet --version` reports `8.26.1` (OpenVoxProject/openvox#415).

#### Known follow-ups (deferred at Wave 1, all resolved in Wave 2)

- ~~Re-capture CLI blocks against current lab~~ — done in Wave 2.
- ~~Verify vardir path~~ — confirmed `/opt/puppetlabs/puppet/cache` is correct in Wave 2; upstream PR pending.
- ~~Bump PostgreSQL recommendation~~ — done in Wave 2.

## [1.0.0] - 2026-03-12

### 🎉 First Stable Release

VoxDocs reaches 1.0 — the OpenVox community documentation is now considered
production-ready. All major sections are complete, technically verified against
the official Puppet 8.10.0 documentation, and suitable for production use.

---

## [0.97.0] - 2026-03-12

### Added

- getting-started/migration.md — **New comprehensive migration guide** for users
  moving from Puppet 7 to OpenVox 8, covering:
  - Strict mode (`strict_variables=true` by default)
  - Legacy facts removed (with complete conversion table)
  - Hiera 3 functions removed (`hiera()` → `lookup()`)
  - Ruby 3.2.x and OpenSSL 3.0 changes
  - Package migration instructions (RHEL/Debian)
  - Complete migration checklist
- module-development/README.md — Added "Linting with openvox-lint" section
  documenting the community linter for style guide violations, legacy facts,
  and deprecated Hiera 3 functions

### Changed

- language/functions.md — **Significantly expanded** from 42 lines to 192 lines:
  - Added String Functions section (case, whitespace, search/replace)
  - Added Array Functions section (unique, flatten, sort, slice)
  - Added Hash Functions section (merge, keys, values, dig)
  - Expanded Hiera Lookup section with signature table and merge strategies
  - Added deprecation warning for hiera()/hiera_array()/hiera_hash()
  - Added Type Checking section
  - Added Conditional Functions section (pick, lest, then)
  - Added Utility Functions section (epp, include, contain, fail, debug)
  - Added See Also cross-references
- getting-started/README.md — Added "Migrating from Puppet 7?" section with
  link to the new migration guide

### Technical Context

- All changes reviewed against official Puppet 8.10.0 documentation (52,477 lines)
- Content verified against Puppet Best Practices (Chris Barbour) and
  Puppet 8 for DevOps Engineers (David Sandilands)

## [0.96.3] - 2026-03-04

### Added

- Added a "Back to ..." navigation link at the bottom of every subpage
  (immediately above the AI disclaimer) in both cli-reference/ (16 files)
  and language/ (14 files) for consistent top-and-bottom navigation

## [0.96.2] - 2026-03-04

### Changed

- language: Split the 926-line README.md into 14 individual topic pages
  following the existing Table of Contents structure
- language/README.md: Rewritten as intro + TOC index with three sections
  (Language Basics, Control Flow & Structure, Advanced Topics)

### Added

- language/resources.md — Resources, titles, and namevars
- language/resource-types.md — file, package, service, user, group, cron, exec
- language/variables.md — Variables, data types, facts, scope
- language/strings.md — Strings, interpolation, heredocs, arrays, hashes
- language/conditionals.md — if/elsif/else, case, selectors
- language/classes.md — Defining, declaring, and parameterizing classes
- language/defined-types.md — Reusable resource templates
- language/relationships.md — Arrows, metaparameters, chaining
- language/functions.md — Built-in functions
- language/templates.md — EPP and ERB templates
- language/node-definitions.md — Node definitions in site.pp
- language/iteration.md — each, map, filter, reduce
- language/regex.md — Regular expressions
- language/example.md — Real-world NTP module example

## [0.96.1] - 2026-03-04

### Changed

- cli-reference: Split the monolithic 1,556-line README.md into 16 individual
  command pages, each with its own .md file
- cli-reference/README.md: Rewritten as a table-of-contents index page with
  three sections (Core Commands, Server & Infrastructure, Additional References)

### Added

- cli-reference/puppet.md — puppet overview (--help, --version)
- cli-reference/puppet-agent.md — puppet agent
- cli-reference/puppet-apply.md — puppet apply
- cli-reference/puppet-resource.md — puppet resource
- cli-reference/puppet-config.md — puppet config
- cli-reference/puppet-ssl.md — puppet ssl
- cli-reference/puppet-module.md — puppet module
- cli-reference/puppet-parser.md — puppet parser
- cli-reference/puppet-lookup.md — puppet lookup
- cli-reference/puppet-subcommands.md — additional subcommands (facts, catalog,
  epp, node, describe, filebucket, device, script, generate)
- cli-reference/facter.md — facter
- cli-reference/puppetserver.md — puppetserver + CA
- cli-reference/r10k.md — r10k (deploy, puppetfile)
- cli-reference/bolt.md — OpenBolt
- cli-reference/puppet-query.md — PuppetDB/PQL queries
- cli-reference/cheatsheet.md — utility commands quick reference

## [0.96.0] - 2026-03-04

### Summary

Second tagged release. Consolidates all corrections, content improvements,
and diagram fixes since the initial v0.92 tag. Documentation is now
genericized for public use (no site-specific references), includes full AI
disclosure on every page, and all ASCII diagrams render correctly.

### Highlights since v0.92

- All domain references use `example.com` (no personal domains)
- Internal IP addresses and personal usernames genericized
- AI disclosure footer added to all 12 documentation pages
- VoxPupuli Community Slack and Connect added to community resources
- PuppetServer description corrected in architecture docs
- Environments section rewritten to match upstream Puppet/OpenVox conventions
- Resource title vs. namevar documentation expanded with examples and reference table
- Related Projects table reorganized (PDK and Onceover removed)
- All ASCII diagrams in architecture and hiera docs aligned and corrected

## [0.95.8] - 2026-03-04

### Fixed

- architecture/README.md: Realigned all boxes in the "How Data Flows" diagram;
  the gaps between boxes were 9 characters but the arrow labels were 10, causing
  Code Dir, PuppetServer, and PuppetDB boxes to shift right on content lines;
  all inter-box gaps are now a consistent 10 characters

## [0.95.7] - 2026-03-04

### Fixed

- architecture/README.md: Aligned the Agent connector ┐, │, and ┴ characters
  vertically so the line from the main diagram connects properly to the
  Agent N box below it

## [0.95.6] - 2026-03-04

### Fixed

- architecture/README.md: Shifted the right-side ┐ of the connector line
  above the three Agent boxes one space right to align with the Agent N box

## [0.95.5] - 2026-03-04

### Fixed

- architecture/README.md: Realigned the right-side outer box border on lines
  15–22 of the Big Picture diagram; the Certificate box content fix in v0.95.4
  shortened those lines by one character, causing the outer │ to misalign

## [0.95.4] - 2026-03-04

### Fixed

- architecture/README.md: Fixed spacing in "The Big Picture" ASCII diagram
  - Removed extra trailing space on the "OpenVox Primary Server" title line
  - Removed extra space after "Certificate", "Authority", and "(SSL/TLS)"
    inside the Certificate Authority box (content was 15 chars, should be 14
    to match the box border dashes)

## [0.95.3] - 2026-03-04

### Fixed

- hiera/README.md: Removed extra trailing space on three lines in the
  "The Three Layers" ASCII diagram (Global Layer, Environment Layer,
  Module Layer) so the box characters align correctly

## [0.95.2] - 2026-03-04

### Changed

- getting-started/README.md: Expanded the Resources section under
  "Understanding the Magic" with a new "A Word About Titles" subsection
  explaining that titles often serve as the resource identity (file path,
  package name), and that when using a descriptive title instead, you must
  explicitly provide the namevar parameter (path, name, etc.)
- language/README.md: Significantly expanded the "Resource Titles vs. Namevar"
  section with clearer explanation of both forms (shorthand vs. explicit),
  parallel examples for file/package/service types, a "Common namevars by type"
  reference table, and a rule-of-thumb callout
- getting-started/README.md: Added cross-reference link to the Language
  Reference for the full namevar explanation

## [0.95.1] - 2026-03-04

### Changed

- community/README.md: Moved openvox-gui and openvox-lint to the bottom of
  the Related Projects table
- community/README.md: Removed PDK and Onceover from Related Projects

## [0.95] - 2026-03-04

### Changed

- architecture/README.md: Rewrote the Environments section to match standard
  Puppet/OpenVox documentation conventions rather than a site-specific setup
- Removed prescriptive "most common setup" table (production/staging/development)
  which reflected one administrator's preference, not the upstream default
- Now correctly documents that only `production` ships as a default environment
- Explains the r10k Git-branch-to-environment workflow with diagram
- Added note about branch name character conversion (e.g. `/` and `-` to `_`)
- Added `site-modules/`, `environment.conf`, and `Puppetfile` to the directory
  tree (matching the standard control repo layout)
- Documents three methods for assigning agents to environments (puppet.conf,
  ENC, command line)

## [0.94] - 2026-03-04

### Fixed

- architecture/README.md: Corrected PuppetServer description item #3 from
  "Serves file content from modules" to "Serves configuration elements from
  modules"

## [0.93] - 2026-03-04

### Added

- AI disclosure footer on all 12 documentation pages for full transparency
- Each page now includes: "This document was created with the assistance of
  AI (Grok, xAI). All technical content has been reviewed and verified by
  human contributors." rendered in small type (`<sub>`) at the bottom

### Files updated

- README.md, getting-started, architecture, language, cli-reference,
  configuration, server-admin, hiera, module-development, orchestration,
  troubleshooting, community

## [0.92] - 2026-03-04

### Added

- community/README.md: Added VoxPupuli Community Slack (voxpupuli.slack.com)
  to Key Community Resources table
- community/README.md: Added VoxPupuli Connect (voxpupuli.org/connect) as a
  hub for all VoxPupuli and OpenVox community links
- community/README.md: Expanded "Where to Ask Questions" section with
  VoxPupuli Slack as the primary recommendation and VoxPupuli Connect as the
  directory of all community channels

### Changed

- community/README.md: Clarified Puppet Community Slack as the broader
  ecosystem channel, distinct from the VoxPupuli-specific Slack

## [0.91] - 2026-03-04

### Changed

- Replaced all domain references with `example.com` across
  all documentation files (README.md, AGENTS.md, cli-reference, configuration)
- Replaced internal IP address `10.0.100.225` with generic `192.168.1.100`
  in CLI reference facter output example
- Replaced personal username `jsheets` with generic `admin` in puppet agent
  disable example
- Fixed broken anchor links in orchestration guide (updated to match renamed
  OpenBolt and versioned r10k section headings)

### Fixed

- AGENTS.md: Updated domain references to use generic `example.com`
- cli-reference/README.md: Genericized 13 domain references, 1 IP address,
  and 1 username across example output and usage patterns
- configuration/README.md: Genericized 6 domain references in puppet.conf
  examples and server configuration section
- orchestration/README.md: Fixed TOC anchor links for renamed sections

## [0.90] - 2026-03-04

### Added

- Initial documentation release with 11 sections
- Complete CLI reference with real command output from a live OpenVox server
- Covers OpenVox Agent 8.25.0, Server 8.12.1, Facter 5.4.0, OpenBolt 5.3.0,
  r10k 5.0.2
- All content is original — no Perforce/Puppet copyright material used
- Licensed under CC BY-SA 4.0
