# 📦 Module Development Guide

> *Building reusable, testable, shareable Puppet code — the right way.*

---

> **Sources:** Module layout, metadata.json, and Forge publish expectations align with
> official OpenVox module docs and [cheatsheet_module](https://docs.openvoxproject.org/openvox/latest/cheatsheet_module.html).
> Roles/profiles opinions and lab habits are community color — [EDITORIAL.md](../EDITORIAL.md).

---

## What's a Module?

A module is a **self-contained bundle of Puppet code** that manages a specific piece of infrastructure. Want to manage Apache? There's a module for that. NTP? Module. Your company's custom monitoring stack? You should write a module for that too.

Modules are the primary way to organize Puppet code. They're shareable (via the [Puppet Forge](https://forge.puppet.com/)), testable, and composable.

---

## Module Structure

Every module follows a standard directory layout:

```text
mymodule/
├── manifests/
│   ├── init.pp              ← Main class (class mymodule { ... })
│   ├── install.pp           ← Package installation
│   ├── config.pp            ← Configuration management
│   └── service.pp           ← Service management
├── files/                    ← Static files (served via puppet:///modules/mymodule/)
│   └── default.conf
├── templates/                ← Dynamic templates (EPP or ERB)
│   └── config.epp
├── lib/
│   ├── facter/              ← Custom facts (Ruby)
│   │   └── myapp_version.rb
│   └── puppet/
│       ├── functions/        ← Custom functions
│       └── types/            ← Custom types
├── data/                     ← Module-level Hiera data
│   ├── common.yaml
│   └── os/
│       ├── RedHat.yaml
│       └── Debian.yaml
├── hiera.yaml                ← Module-level Hiera config
├── spec/                     ← Tests
│   ├── spec_helper.rb
│   └── classes/
│       └── init_spec.rb
├── examples/                 ← Usage examples
│   └── init.pp
├── CHANGELOG.md
├── README.md
└── metadata.json             ← Module metadata
```

### The Name Matters

Module names follow the pattern `author-modulename`. The directory on disk is just `modulename` (without the author prefix). Class names map directly to file paths:

| Class Name | File Path |
|-----------|-----------|
| `mymodule` | `manifests/init.pp` |
| `mymodule::install` | `manifests/install.pp` |
| `mymodule::config` | `manifests/config.pp` |
| `mymodule::config::ssl` | `manifests/config/ssl.pp` |

---

## The Roles and Profiles Pattern

The most widely-used pattern for organizing Puppet code at scale. If you learn one pattern from this guide, make it this one.

### Profiles

Profiles are **technology-specific wrappers** around modules. They configure a single piece of technology:

```puppet
# site-modules/profile/manifests/webserver.pp
class profile::webserver (
  Integer $port = 80,
  Boolean $ssl  = false,
) {
  class { 'apache':
    default_vhost => false,
    mpm_module    => 'prefork',
  }

  apache::vhost { $facts['networking']['fqdn']:
    port    => $port,
    docroot => '/var/www/html',
    ssl     => $ssl,
  }

  if $ssl {
    include apache::mod::ssl
  }
}

# site-modules/profile/manifests/database.pp
class profile::database (
  String $root_password = lookup('profile::database::root_password'),
) {
  class { 'mysql::server':
    root_password    => $root_password,
    override_options => {
      'mysqld' => {
        'max_connections' => 200,
        'bind-address'    => '0.0.0.0',
      },
    },
  }
}

# site-modules/profile/manifests/base.pp
class profile::base {
  include ntp
  include ssh
  include firewall
  include monitoring
}
```

### Roles

Roles define **what a server IS** by composing profiles. Each server gets exactly **one** role. A role should be nothing but `include` statements:

```puppet
# site-modules/role/manifests/webserver.pp
class role::webserver {
  include profile::base
  include profile::webserver
  include profile::monitoring
}

# site-modules/role/manifests/database.pp
class role::database {
  include profile::base
  include profile::database
  include profile::monitoring
  include profile::backup
}

# site-modules/role/manifests/app_server.pp
class role::app_server {
  include profile::base
  include profile::webserver
  include profile::database
  include profile::app
}
```

### Assigning Roles

In `site.pp` or via Hiera/ENC:

```puppet
# manifests/site.pp
node default {
  include role::base
}

node /^web\d+/ {
  include role::webserver
}

node /^db\d+/ {
  include role::database
}
```

Or via Hiera (cleaner for large fleets):

```yaml
# data/nodes/web01.example.com.yaml
classes:
  - role::webserver
```

```puppet
# manifests/site.pp
lookup('classes', Array[String], 'unique', []).include
```

> **Best Practice:** For fleets larger than ~10 nodes, avoid regex node definitions in `site.pp`. Use Hiera or an ENC (External Node Classifier) to assign roles. This keeps your `site.pp` clean and makes role assignments data-driven. Also, remember: **one role per node**. Roles compose profiles, which compose modules. This hierarchy keeps your code maintainable and your mental model clear.

---

## Writing Custom Facts

Custom facts extend Facter with organization-specific information.

### Ruby Facts

```ruby
# lib/facter/myapp_version.rb
Facter.add(:myapp_version) do
  confine kernel: 'Linux'
  setcode do
    if File.exist?('/opt/myapp/VERSION')
      File.read('/opt/myapp/VERSION').strip
    else
      'not installed'
    end
  end
end
```

### Structured Facts

```ruby
# lib/facter/myapp_info.rb
Facter.add(:myapp_info) do
  setcode do
    result = {}
    if File.exist?('/opt/myapp/VERSION')
      result['version'] = File.read('/opt/myapp/VERSION').strip
      result['installed'] = true
      result['config_dir'] = '/etc/myapp'
    else
      result['installed'] = false
    end
    result
  end
end
```

Access in Puppet: `$facts['myapp_info']['version']`

### External Facts (No Ruby Required!)

Drop a script or YAML file in `/etc/puppetlabs/facter/facts.d/`:

```yaml
# /etc/puppetlabs/facter/facts.d/datacenter.yaml
---
datacenter: us-east-1
rack: A42
environment_type: production
```

Or a script:

```bash
#!/bin/bash
# /etc/puppetlabs/facter/facts.d/hardware.sh
echo "chassis_type=$(dmidecode -s chassis-type 2>/dev/null || echo unknown)"
echo "bios_vendor=$(dmidecode -s bios-vendor 2>/dev/null || echo unknown)"
```

---

## metadata.json

Every module needs a `metadata.json`:

```json
{
  "name": "myorg-webserver",
  "version": "1.0.0",
  "author": "My Organization",
  "summary": "Manages Apache web server configuration",
  "license": "Apache-2.0",
  "source": "https://github.com/myorg/puppet-webserver",
  "dependencies": [
    { "name": "puppetlabs-apache", "version_requirement": ">= 8.0.0 < 13.0.0" },
    { "name": "puppetlabs-stdlib", "version_requirement": ">= 8.0.0 < 10.0.0" }
  ],
  "operatingsystem_support": [
    { "operatingsystem": "RedHat", "operatingsystemrelease": ["8", "9", "10"] },
    { "operatingsystem": "Debian", "operatingsystemrelease": ["11", "12"] },
    { "operatingsystem": "Ubuntu", "operatingsystemrelease": ["22.04", "24.04"] }
  ],
  "requirements": [
    { "name": "puppet", "version_requirement": ">= 7.0.0 < 9.0.0" }
  ]
}
```

---

## Testing Your Modules

### Unit Tests with rspec-puppet

```ruby
# spec/classes/init_spec.rb
require 'spec_helper'

describe 'webserver' do
  on_supported_os.each do |os, os_facts|
    context "on #{os}" do
      let(:facts) { os_facts }

      it { is_expected.to compile.with_all_deps }
      it { is_expected.to contain_package('httpd') }
      it { is_expected.to contain_service('httpd').with_ensure('running') }

      context 'with ssl enabled' do
        let(:params) { { ssl: true } }
        it { is_expected.to contain_class('apache::mod::ssl') }
      end

      context 'with custom port' do
        let(:params) { { port: 8080 } }
        it { is_expected.to contain_apache__vhost(os_facts[:fqdn]).with_port(8080) }
      end
    end
  end
end
```

### Running Tests

```bash
# Install test dependencies
bundle install

# Run unit tests
bundle exec rake spec

# Run a specific test file
bundle exec rspec spec/classes/init_spec.rb

# Validate syntax
bundle exec rake validate

# Run lint checks
bundle exec rake lint
```

### Linting

Use the ecosystem linters that match your toolchain:

- **[puppet-lint](https://github.com/puppetlabs/puppet-lint)** (and its Vox Pupuli plugins) for style and common anti-patterns
- Module **PDK / [jig](https://github.com/voxpupuli/jig)** validate targets when you build with those tools
- Prefer structured facts (`$facts['os']['family']`) over legacy top-level facts — official OpenVox/Puppet style

```bash
# Typical gem-based lint in a module
bundle exec rake lint
# or
puppet-lint manifests/
```

> **Pro tip:** Run lint in CI so arrow-alignment crimes never reach production.

### Acceptance Tests with Litmus

For integration testing against real systems:

```bash
# Provision a test node
bundle exec rake 'litmus:provision[docker, litmusimage/centos:9]'

# Install the module on the test node
bundle exec rake litmus:install_agent
bundle exec rake litmus:install_module

# Run acceptance tests
bundle exec rake litmus:acceptance:parallel

# Tear down
bundle exec rake litmus:tear_down
```

---

## Publishing to the Forge

```bash
# Build the module package
puppet module build

# This creates a .tar.gz in the pkg/ directory
# Upload it to https://forge.puppet.com/upload
```

### PDK and Open-Source Alternatives

The **Puppet Development Kit (PDK)** automates module scaffolding, validation, and testing. Puppet (Perforce) ceased open-source maintenance at PDK **3.4.0**; newer versions require a commercial Puppet Forge API key or PE license.

PDK 3.4.0 remains compatible with OpenVox for module development. If you have it installed:

```bash
pdk new module myorg-newmodule
pdk new class install
pdk validate
pdk test unit
pdk build
```

The community maintains two open-source alternatives that do not require a Puppet account:

- **[VoxBox](https://github.com/voxpupuli/container-voxbox)** -- A container image maintained by Vox Pupuli that includes rspec-puppet, Litmus, RuboCop, and other testing gems. It is the recommended way to run unit and acceptance tests for OpenVox modules in CI and local development.
- **[jig](https://github.com/avitacco/jig)** -- A Go-based reimplementation of PDK. Ships as a single static binary with no Ruby runtime dependency and supports module scaffolding, building, and releasing.

---

## Best Practices Checklist

- [ ] Follow the **roles and profiles** pattern
- [ ] Use **Hiera** for all parameter data (not hardcoded values)
- [ ] Include **type validation** for all class parameters
- [ ] Write **unit tests** for every class and defined type
- [ ] Use **EPP templates** (not ERB) for new code
- [ ] Include a clear **README.md** with usage examples
- [ ] Maintain a **CHANGELOG.md**
- [ ] Pin module **dependencies** in `metadata.json`
- [ ] Use **`contain`** instead of `include` for classes that need ordering
- [ ] Never use **`exec`** when a proper resource type exists

---

*Next up: [Orchestration →](../orchestration/README.md)*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
