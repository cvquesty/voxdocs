# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
- Replaced all `questy.org` domain references with `example.com` across
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
