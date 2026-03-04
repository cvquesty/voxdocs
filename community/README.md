# 🤝 Community & Contributing

> *Come for the automation, stay for the people.*

---

## The OpenVox Community

OpenVox exists because of its community. Born from the [Vox Pupuli](https://voxpupuli.org/) collective — a group of Puppet module maintainers who've been collaborating for years — OpenVox carries forward the tradition of open-source infrastructure management.

### Key Community Resources

| Resource | URL | Purpose |
|----------|-----|---------|
| **OpenVox GitHub** | [github.com/openvoxproject](https://github.com/openvoxproject) | Source code, issues, PRs |
| **Vox Pupuli** | [voxpupuli.org](https://voxpupuli.org/) | Community hub, blog, docs |
| **Puppet Community Slack** | [puppetcommunity.slack.com](https://puppetcommunity.slack.com/) | Real-time chat |
| **Puppet Forge** | [forge.puppet.com](https://forge.puppet.com/) | Module repository |
| **Vox Pupuli Blog** | [voxpupuli.org/blog](https://voxpupuli.org/blog/) | Release announcements, articles |

### Where to Ask Questions

1. **Puppet Community Slack** — Best for quick questions and real-time discussion. The `#openvox` and `#voxpupuli` channels are great places to start.
2. **GitHub Issues** — For bug reports, feature requests, and technical discussions on specific repos.
3. **Stack Overflow** — Tag your questions with `puppet` and `openvox`.
4. **Vox Pupuli Mailing Lists** — For broader community discussions.

---

## How to Contribute

### Reporting Bugs

Found a bug? Awesome (well, not awesome for you, but we appreciate the report). Here's how:

1. **Search first** — check existing issues on the relevant GitHub repo
2. **Create an issue** with:
   - OpenVox/Puppet version (`puppet --version`)
   - OS and version (`facter os.release.full`)
   - Steps to reproduce
   - Expected vs. actual behavior
   - Relevant log output (from `puppet agent -t --debug` or server logs)

### Contributing Code

1. **Fork** the repository on GitHub
2. **Create a branch** for your change (`git checkout -b fix/ssl-error-handling`)
3. **Write tests** for your change
4. **Follow the style** of the existing codebase
5. **Submit a pull request** with a clear description

### Contributing Modules

The Puppet Forge welcomes modules from anyone! If you've built something useful:

1. Follow the [Module Development Guide](../module-development/README.md)
2. Include tests (rspec-puppet at minimum)
3. Write a clear README with usage examples
4. Publish to the Forge

### Contributing to These Docs

This documentation lives at [github.com/cvquesty/voxdocs](https://github.com/cvquesty/voxdocs) (if published). Contributions are welcome! Fix a typo, improve an explanation, add an example — every bit helps.

---

## Related Projects

The Puppet/OpenVox ecosystem is vast. Here are some projects worth knowing about:

| Project | Description | Link |
|---------|-------------|------|
| **openvox-gui** | Web-based management GUI for OpenVox | [github.com/cvquesty/openvox-gui](https://github.com/cvquesty/openvox-gui) |
| **openvox-lint** | Puppet manifest linter (modernized puppet-lint) | [rubygems.org/gems/openvox-lint](https://rubygems.org/gems/openvox-lint) |
| **Puppet Forge** | Community module repository | [forge.puppet.com](https://forge.puppet.com/) |
| **PDK** | Puppet Development Kit | [puppet.com/docs/pdk](https://puppet.com/docs/pdk/) |
| **Onceover** | Testing tool for control repos | [github.com/voxpupuli/onceover](https://github.com/voxpupuli/onceover) |
| **puppet-lint** | Original Puppet linter | [github.com/puppetlabs/puppet-lint](https://github.com/puppetlabs/puppet-lint) |
| **rspec-puppet** | Unit testing for Puppet | [github.com/puppetlabs/rspec-puppet](https://github.com/puppetlabs/rspec-puppet) |
| **Litmus** | Acceptance testing framework | [github.com/puppetlabs/puppet_litmus](https://github.com/puppetlabs/puppet_litmus) |
| **puppetboard** | PuppetDB web frontend | [github.com/voxpupuli/puppetboard](https://github.com/voxpupuli/puppetboard) |

---

## Conferences and Events

- **VoxConf** — Vox Pupuli's community conference ([voxpupuli.org/voxconf](https://voxpupuli.org/voxconf/))
- **Config Management Camp** — Annual configuration management conference in Ghent, Belgium
- **PuppetCamp** — Regional community meetups

---

## Code of Conduct

The OpenVox and Vox Pupuli communities are committed to providing a welcoming, inclusive environment for everyone. Be kind, be respectful, and be constructive. We're all here because we love automating things and occasionally complaining about YAML indentation.

---

## Acknowledgments

This project stands on the shoulders of giants:

- **Puppet Labs** (now Perforce) — for creating the original Puppet platform and open-sourcing it
- **Vox Pupuli** — for maintaining hundreds of community modules and creating OpenVox
- The **docs-archive** — for preserving the legacy Puppet documentation
- **Overlook InfraTech** and **betadots GmbH** — for sponsoring OpenVox development
- **The entire Puppet community** — decades of contributions, modules, blog posts, and Stack Overflow answers

And of course, thanks to **you** for reading this far. Now go automate something! 🦊

---

*You've reached the end of the docs. Go back to the [Documentation Map](../README.md) or start managing some infrastructure!*
