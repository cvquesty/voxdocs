# VoxDocs Project Instructions

## Writing Style

- Conversational, helpful tone — like explaining to a smart friend
- Target audience: college senior level
- Light humor is encouraged (but never at the reader's expense)
- Use Markdown features extensively: tables, code blocks, admonitions, links, emoji
- Every CLI example should be copy-pasteable
- All binary references must include **actual command-line output** from a sample OpenVox server

## Current Verified Versions (from sample infrastructure)

- OpenVox Agent: 8.26.2 (released 2026-04-18; binary reports 8.26.1 due to openvox#415)
- OpenVox Server (PuppetServer): 8.12.1
- OpenFact (was Facter): 5.6.0 (released 2026-04-09)
- r10k: 5.0.2
- OpenBolt: 5.4.0 (released 2026-03-04)
- OpenVoxDB: 8.8.1
- Server OS: RHEL 9.7

> **Branding note:** The OpenVox project is rebranding the platform's components.
> Facter is now **OpenFact**; PuppetDB is now **OpenVoxDB**; Bolt is now **OpenBolt**.
> Binary names (`facter`, `puppetdb`, `bolt`) and config file names (`facter.conf`)
> are unchanged. Only the project/package names have been rebranded.

## Best Practices to Emphasize Throughout

- Roles and profiles pattern for code organization at scale
- Class containment (`contain`, `require`) to manage dependencies properly
- Idempotent execs with `creates`, `onlyif`, `unless`, `refreshonly`
- Trailing commas in resource attributes (cleaner Git diffs, fewer syntax errors)
- Use `include` with Hiera over resource-like class declarations when possible
- Proper use of facts: `$facts['os']['family']` not legacy `$operatingsystem`
- Node definitions only for small sites; use ENC or Hiera for larger deployments
- Use EPP templates over ERB when possible (better performance, cleaner syntax)
- Keep manifests under 50 lines; refactor into classes/defined types
- Always document your modules with README.md and examples/

## Structure

- Each major section has its own directory with a `README.md`
- Cross-reference between docs using relative links
- Code blocks must specify the language for syntax highlighting
- Use `> **Note:**` and `> **Warning:**` for callouts
- Emoji section headers for visual scanning

## Copyright Policy

- NEVER copy text from Perforce/Puppet's official documentation at puppet.com
- Reference the docs-archive (Puppet 5.x, unencumbered) for structure inspiration only
- The official OpenVox docs (github.com/OpenVoxProject/openvox-docs, CC BY-SA 3.0, copyright Puppet, Inc.) may be referenced for factual data: version numbers, platform support, component names, release dates, and known issues
- All explanatory text, examples, and prose must be originally written
- When describing Puppet language features, use our own examples and explanations
- All CLI output must be captured from our own infrastructure

## Conventional Commits

- Use conventional commits for all changes
- Default branch: `development`
