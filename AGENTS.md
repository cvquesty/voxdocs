# VoxDocs Project Instructions

## Writing Style
- Conversational, helpful tone — like explaining to a smart friend
- Target audience: college senior level
- Light humor is encouraged (but never at the reader's expense)
- Use Markdown features extensively: tables, code blocks, admonitions, links, emoji
- Every CLI example should be copy-pasteable
- All binary references must include **actual command-line output** from a sample OpenVox server

## Current Verified Versions (from sample infrastructure)
- OpenVox Agent: 8.25.0
- OpenVox Server (PuppetServer): 8.12.1
- Facter: 5.4.0
- r10k: 5.0.2
- OpenBolt: 5.3.0
- Server OS: RHEL 9.7

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
- All content must be originally written
- When describing Puppet language features, use our own examples and explanations
- All CLI output must be captured from our own infrastructure

## Conventional Commits
- Use conventional commits for all changes
- Default branch: `development`
