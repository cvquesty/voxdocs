# VoxDocs Project Instructions

## Writing Style
- Conversational, helpful tone — like explaining to a smart friend
- Target audience: college senior level
- Light humor is encouraged (but never at the reader's expense)
- Use Markdown features extensively: tables, code blocks, admonitions, links, emoji
- Every CLI example should be copy-pasteable
- Prefer **“a live lab”** / `*.example.com` — never publish real lab FQDNs (e.g. personal `*.questy.org` hosts)

## Canonical facts vs this site

Full merge rules: **[EDITORIAL.md](EDITORIAL.md)**.
- **[docs.openvoxproject.org](https://docs.openvoxproject.org/)** (OpenVoxProject/openvox-docs) is **canonical for product facts**: component versions, platform support, naming, install prerequisites
- This repo keeps its **own design, Docsify shell, structure, and voice**
- When facts diverge, **update this repo to match official** — do not overwrite our design with their site
- Weekly job should be a **content audit**, not a full-site mirror

## Current Latest Shipping (canonical check 2026-07-22)
Cross-checked with official docs SBOM tables + GitHub releases:

- openvox-agent / OpenVox agent line: **8.28.1**
- openvox-server: **8.15.0**
- openvoxdb / openvoxdb-termini: **8.15.0**
- openbolt: **5.6.0** (bundles r10k **5.0.3**)
- OpenFact: **5.7.0** standalone; recent agents bundle **5.6.1+**
- r10k: **not** in openvox-server/agent packages — OpenBolt is the only OpenVox package that bundles it

## Lab Verified Snapshot (2026-05-23) — CLI capture source
- OpenVox Agent: 8.26.2 (package; `puppet --version` reports 8.26.1 due to openvox#415)
- OpenVox Server: 8.12.1
- OpenFact: 5.6.0
- r10k: 5.0.2
- OpenBolt: 5.4.0 (package `openbolt`)
- OpenVoxDB: 8.12.1 (+ termini)
- Server OS: RHEL 9.7

> **Note:** Lab CLI captures lag absolute latest on purpose so pasted output stays real. Always show **Latest Shipping** vs **Lab Verified** when both matter.

## Structure
- Each major section has its own directory with a `README.md`
- Docsify shell: `index.html`, `_sidebar.md`, `_navbar.md`
- Cross-reference between docs using relative links
- Code blocks must specify the language for syntax highlighting
- Use `> **Note:**` and `> **Warning:**` for callouts
- Emoji section headers for visual scanning

## Copyright Policy
- NEVER copy text from Perforce/Puppet's official documentation at puppet.com
- The official OpenVox docs (github.com/OpenVoxProject/openvox-docs, CC BY-SA 3.0) may be referenced for factual data: version numbers, platform support, component names, release dates, and known issues
- Reference the docs-archive (Puppet 5.x, unencumbered) for structure inspiration only
- All narrative content must be originally written
- When describing Puppet language features, use our own examples and explanations
- All CLI output must be captured from our own lab infrastructure (genericized hostnames)

## Conventional Commits
- Use conventional commits for all changes
- Default branch: `development`
