# VoxDocs Project Instructions

## Writing Style
- Conversational, helpful tone — like explaining to a smart friend
- Target audience: college senior level
- Light humor is encouraged (but never at the reader's expense)
- Use Markdown features extensively: tables, code blocks, admonitions, links, emoji
- Every CLI example should be copy-pasteable
- All binary references must include **actual command-line output** from openvox.questy.org

## Current Verified Versions (from openvox.questy.org)
- OpenVox Agent: 8.25.0
- OpenVox Server (PuppetServer): 8.12.1
- Facter: 5.4.0
- r10k: 5.0.2
- OpenBolt: 5.3.0
- Server OS: RHEL 9.7

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
