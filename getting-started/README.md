# 🚀 Getting Started with OpenVox

> *"The journey of a thousand servers begins with a single `puppet apply`."*
> — Ancient DevOps Proverb (okay, we made that up)

---

## Welcome!

So you want to manage infrastructure with code? Excellent life choice. Whether you're setting up 3 servers or 3,000, OpenVox has your back. This guide will take you from zero to "Hey, it actually works!" in about 30 minutes.

**What you'll learn:**
- [Installing OpenVox](#installation)
- [Your first manifest](#your-first-manifest)
- [Understanding what just happened](#understanding-the-magic)
- [Connecting an agent to a server](#agent-server-setup)
- [Where to go next](#next-steps)

---

## Prerequisites

Before we dive in, you'll need:

- A Linux server (RHEL 8/9/10, Rocky, AlmaLinux, Ubuntu 22.04+, Debian 11+, or Fedora 42+)
- Root or `sudo` access
- Internet connectivity (to reach the package repos)
- A healthy sense of adventure

> **Note:** OpenVox also supports macOS, Windows, SLES, and Amazon Linux. This guide focuses on the RHEL/Debian families because that's where most of the action is.

---

## Installation

### Step 1: Install the OpenVox Repository

OpenVox packages live in the Vox Pupuli repositories. First, let's add the appropriate repo for your platform.

#### RHEL / CentOS / Rocky / AlmaLinux / Fedora

```bash
# Install the OpenVox 8 release repository
sudo rpm -Uvh https://yum.voxpupuli.org/openvox8-release-el-$(rpm -E %{rhel}).noarch.rpm
```

#### Debian / Ubuntu

```bash
# Download and install the OpenVox 8 release package
wget https://apt.voxpupuli.org/openvox8-release-$(lsb_release -cs).deb
sudo dpkg -i openvox8-release-$(lsb_release -cs).deb
sudo apt-get update
```

### Step 2: Install the OpenVox Agent

The agent is the workhorse — it's what actually applies configuration to your system.

#### RHEL Family

```bash
sudo yum install -y openvox-agent
```

#### Debian Family

```bash
sudo apt-get install -y openvox-agent
```

### Step 3: Verify the Installation

After installation, the `puppet` binary should be available on your path:

```bash
# Check the version
puppet --version
```

Expected output:
```
8.25.0
```

> **Pro tip:** The OpenVox agent installs into `/opt/puppetlabs/`. The binary lives at `/opt/puppetlabs/bin/puppet`. The installer adds this to your PATH, but if you're in a weird shell, you may need to source your profile or use the full path.

### Step 4: (Optional) Install the OpenVox Server

If you want a central server to manage multiple nodes (and you probably do), install the server package on your designated primary node:

#### RHEL Family

```bash
sudo yum install -y openvox-server
sudo systemctl enable --now openvox-server
```

#### Debian Family

```bash
sudo apt-get install -y openvox-server
sudo systemctl enable --now openvox-server
```

Verify the server is running:

```bash
sudo systemctl status openvox-server
```

---

## Your First Manifest

Let's write some infrastructure-as-code! A **manifest** is a file (ending in `.pp`) that describes the desired state of your system using the Puppet language.

### Hello, OpenVox!

Create a file called `hello.pp`:

```puppet
# hello.pp — Your first OpenVox manifest!

# This ensures a file exists with specific content
file { '/tmp/hello-openvox.txt':
  ensure  => file,
  content => "Hello from OpenVox! 🦊\nManaged by Puppet DSL.\n",
  mode    => '0644',
}

# Let's also make sure a useful package is installed
package { 'tree':
  ensure => installed,
}

# And print a friendly notification
notify { 'welcome_message':
  message => 'OpenVox is now managing this system. Resistance is futile (but also unnecessary).',
}
```

### Apply It!

```bash
sudo puppet apply hello.pp
```

Expected output:

```
Notice: Compiled catalog for myhost.example.com in environment production in 0.12 seconds
Notice: /Stage[main]/Main/File[/tmp/hello-openvox.txt]/ensure: defined content as '{sha256}abc123...'
Notice: /Stage[main]/Main/Package[tree]/ensure: created
Notice: OpenVox is now managing this system. Resistance is futile (but also unnecessary).
Notice: Applied catalog in 2.45 seconds
```

### Verify It Worked

```bash
cat /tmp/hello-openvox.txt
```

```
Hello from OpenVox! 🦊
Managed by Puppet DSL.
```

```bash
which tree
```

```
/usr/bin/tree
```

🎉 **Congratulations!** You just used infrastructure-as-code to manage your system's state. The file was created, the package was installed, and the notification was printed. If you run `puppet apply hello.pp` again, nothing will change — because the system already matches the desired state. That's **idempotence**, and it's the secret sauce of configuration management.

---

## Understanding the Magic

Let's break down what just happened:

### Resources

Everything in Puppet/OpenVox is a **resource**. A resource is a single unit of configuration — a file, a package, a service, a user, a cron job. Each resource has:

- A **type** (what kind of thing: `file`, `package`, `service`, etc.)
- A **title** (a unique name for this resource)
- **Attributes** (the desired properties: `ensure`, `content`, `mode`, etc.)

```puppet
# Anatomy of a resource
type { 'title':
  attribute => value,
  another   => value,
}
```

### Idempotence

Run the same manifest 100 times and you get the same result. Puppet doesn't blindly execute commands — it checks the current state, compares it to the desired state, and only makes changes when something is out of spec. This means you can safely run Puppet over and over without fear of breaking things.

### The Catalog

When you run `puppet apply`, the Puppet compiler reads your manifest and builds a **catalog** — a complete description of all the resources and their relationships. The catalog is then applied to the system. Think of it as a blueprint: Puppet reads the blueprint, looks at the building, and fixes anything that doesn't match.

---

## Agent-Server Setup

Using `puppet apply` is great for standalone work, but the real power of OpenVox comes from the agent-server architecture, where a central **Primary Server** compiles catalogs for all your nodes.

### On the Server

1. Install the server (see [installation](#step-4-optional-install-the-openvox-server) above)
2. The CA (Certificate Authority) is automatically configured
3. The server listens on port **8140** by default

### On Each Agent Node

1. Install the agent package
2. Configure the agent to point to your server:

```bash
sudo puppet config set server your-openvox-server.example.com --section agent
```

3. Run the agent once to request a certificate:

```bash
sudo puppet agent -t
```

You'll see something like:

```
Info: Creating a new RSA SSL key for agent1.example.com
Info: csr_attributes file loading from /etc/puppetlabs/puppet/csr_attributes.yaml
Info: Creating a new SSL certificate request for agent1.example.com
Info: Certificate Request fingerprint (SHA256): AB:CD:12:34:...
Exiting; no certificate found and waitforcert is disabled
```

### Back on the Server

Sign the agent's certificate:

```bash
sudo puppetserver ca sign --certname agent1.example.com
```

Or sign all pending requests:

```bash
sudo puppetserver ca sign --all
```

### Back on the Agent

Run the agent again:

```bash
sudo puppet agent -t
```

This time it should successfully connect, download its catalog, and apply it. You're in business!

### Enable the Agent Service

To have the agent run automatically every 30 minutes:

```bash
sudo systemctl enable --now puppet
```

> **Pro tip:** You can change the run interval in `puppet.conf`:
> ```ini
> [agent]
> runinterval = 1h
> ```

---

## Quick Reference Card

Here are the commands you'll use most often as you're getting started:

| Command | What It Does |
|---------|-------------|
| `puppet --version` | Shows the installed version |
| `puppet apply manifest.pp` | Applies a local manifest |
| `puppet apply --noop manifest.pp` | Dry-run (shows what *would* change) |
| `puppet agent -t` | One-time agent run (test mode) |
| `puppet resource user` | Lists all users on the system |
| `puppet resource package httpd` | Shows the state of the `httpd` package |
| `puppet config print all` | Dumps all configuration settings |
| `puppet module list` | Lists installed modules |
| `facter os.name` | Shows the OS name fact |
| `facter --json` | Dumps all facts as JSON |

---

## Next Steps

Now that you've got OpenVox up and running, here's where to go next:

1. **[Architecture & Concepts](../architecture/README.md)** — Understand how all the pieces fit together
2. **[The Puppet Language](../language/README.md)** — Learn the full DSL (classes, defined types, conditionals, and more)
3. **[Configuration Reference](../configuration/README.md)** — Master `puppet.conf` and all the knobs you can turn
4. **[CLI Reference](../cli-reference/README.md)** — The complete guide to every binary and every flag

---

*Next up: [Architecture & Concepts →](../architecture/README.md)*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
