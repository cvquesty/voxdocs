# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-03-12

### 🎉 First Stable Release!

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
