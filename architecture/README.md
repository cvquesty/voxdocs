# 🏗️ Architecture & Concepts

> *Understanding how all the pieces fit together — or, "Why are there so many services?"*

---

> **Sources:** Product facts (catalog stages, agent–server vs apply, component roles)
> follow the official [OpenVox architecture overview](https://docs.openvoxproject.org/openvox/latest/architecture.html).
> Diagrams, lab war stories, and snark remain community-written. See [EDITORIAL.md](../EDITORIAL.md).

---

## The Big Picture

OpenVox can configure systems in two ways (official model):

1. **Agent–server** — managed nodes run the **OpenVox agent**; one or more hosts run **OpenVox server** (catalog compiler + CA). Agents check in over HTTPS (port **8140**, mTLS).
2. **Stand-alone apply** — `puppet apply` compiles and applies a catalog on the same machine (great for labs and golden images; less ideal for large fleets).

In either case, OpenVox is **declarative**: you describe the **desired state**, not a shell script of verbs. The platform compiles a **catalog** (the blueprint for one node), then **applies** it. Here's the 30,000-foot view of the agent–server layout:

```text
┌─────────────────────────────────────────────────────┐
│                  OpenVox Primary Server             │
│                                                     │
│  ┌──────────────┐  ┌──────────┐  ┌──────────────┐   │
│  │ PuppetServer │  │ PuppetDB │  │ Certificate  │   │
│  │  (Catalog    │  │ (Facts,  │  │ Authority    │   │
│  │   Compiler)  │  │ Reports, │  │ (SSL/TLS)    │   │
│  │              │  │ Resources│  │              │   │
│  └──────┬───────┘  └────┬─────┘  └──────┬───────┘   │
│         │               │               │           │
│         └───────────────┼───────────────┘           │
│                         │                           │
│  ┌──────────────────────┴────────────────────────┐  │
│  │        Code Directory (/etc/puppetlabs/code)  │  │
│  │  environments/ → production/ → manifests/     │  │
│  │                              → modules/       │  │
│  │                              → data/ (Hiera)  │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────┬───────────────────────────────┘
                      │ Port 8140 (HTTPS/mTLS)
          ┌───────────┼──────────┐
          │           │          │
    ┌─────┴─────┐ ┌───┴─────┐ ┌──┴──────┐
    │  Agent 1  │ │ Agent 2 │ │ Agent N │
    │  (node1)  │ │ (node2) │ │ (nodeN) │
    └───────────┘ └─────────┘ └─────────┘
```

---

## Core Components

> **Naming:** Official docs say **OpenVox agent**, **OpenVox server**, and **OpenVoxDB**.
> Packages are `openvox-agent`, `openvox-server`, `openvoxdb`. For compatibility the
> binaries and systemd units often still say `puppet`, `puppetserver`, and `puppetdb`.
> Both vocabularies appear below on purpose.

### 🦊 OpenVox agent (`puppet` / package `openvox-agent`)

The agent runs on **every managed node**. Officially it:

1. **Gathers facts** (via **OpenFact**, binary still `facter`)
2. **Sends facts** and requests a catalog from OpenVox server
3. **Applies the catalog** — converges the node to desired state (or reports would-be changes in no-op)
4. **Sends a report** back to the server

Default check-in is every **30 minutes** (`runinterval`). Manual run: `puppet agent -t`.

**Key paths:**

| Path | Purpose |
|------|---------|
| `/opt/puppetlabs/bin/puppet` | Public CLI entrypoint |
| `/opt/puppetlabs/puppet/bin/` | Internal tools (`facter`, bundled Ruby gems) |
| `/etc/puppetlabs/puppet/puppet.conf` | Agent configuration |
| `/opt/puppetlabs/puppet/cache/` | Cached catalogs, facts, reports |
| `/etc/puppetlabs/puppet/ssl/` | SSL certificates |

### 🖥️ OpenVox server (`puppetserver` / package `openvox-server`)

OpenVox server is the **catalog compiler** and usually the **CA**. It's a JVM app (Clojure + JRuby on Jetty) that:

1. **Receives agent requests** over HTTPS (port 8140)
2. **Compiles catalogs** from code + facts
3. **Serves files** and plugins from modules
4. **Manages certificates** (unless you split the CA)
5. **Talks to OpenVoxDB** when termini are configured

**Official fact:** a supported **JDK (17 or 21)** is **not** bundled — you install Java yourself before or with the server.

Yes, it's Java wrapping Ruby. No, we don't lead with that at parties.

**Key paths:**

| Path | Purpose |
|------|---------|
| `/opt/puppetlabs/bin/puppetserver` | The server binary |
| `/etc/puppetlabs/puppetserver/` | Server configuration |
| `/etc/puppetlabs/puppetserver/conf.d/` | Server config fragments |
| `/var/log/puppetlabs/puppetserver/` | Server logs |

### 📊 OpenVoxDB (package `openvoxdb`, unit `puppetdb`)

**OpenVoxDB** is the data warehouse. On each successful run it stores:

- **Facts** — inventory of each node
- **Catalogs** — desired state
- **Reports** — what changed
- **Resources** — including exported/collected resources

Packages: `openvoxdb` + `openvoxdb-termini` (termini version matches OpenVoxDB). The systemd unit remains `puppetdb`.

**Official fact:** OpenVoxDB needs **PostgreSQL 11+** (project recommends **14+**). PostgreSQL is **not** inside the OpenVoxDB package.

Want every CentOS node with >16 GB RAM? **PQL** can do that in one line:

```text
nodes[certname] { facts.os.name = "CentOS" and facts.memory.system.total_bytes > 17179869184 }
```

**Key paths:**

| Path | Purpose |
|------|---------|
| `/etc/puppetlabs/puppetdb/` | PuppetDB configuration |
| `/etc/puppetlabs/puppetdb/conf.d/` | Config fragments |
| Port 8081 | PuppetDB API (HTTPS with mTLS) |

### 🔐 Certificate Authority (CA)

OpenVox uses **mutual TLS** (mTLS) for all communication between agents and the server. This means both sides present certificates — the server proves it's the real server, and the agent proves it's a real agent. The CA (built into PuppetServer) manages this:

1. Agent generates a key pair and sends a **Certificate Signing Request** (CSR)
2. An admin (or autosign policy) **signs the certificate**
3. Both sides now trust each other via the CA's root certificate

This is PKI (Public Key Infrastructure) done right. Every node gets its own certificate, and compromised nodes can be revoked individually.

### 📋 Facter

Facter is a **cross-platform system profiling tool**. It discovers facts about the node — things like:

- Operating system and version
- IP addresses and MAC addresses
- CPU count, architecture, and model
- Memory and disk information
- Cloud provider metadata (AWS, GCP, Azure)
- Virtualization status

Facts are available in your Puppet code as variables (e.g., `$facts['os']['name']`), which lets you write conditional logic like "install Apache on RedHat, install apache2 on Debian."

> **Branding note:** Facter has been rebranded to **OpenFact** in the OpenVox ecosystem. The `facter` binary, `facter.conf` configuration file, and the `facts.d/` directory all keep their existing names — only the project/product name has changed. Standalone OpenFact is at **5.7.0+**; recent agents bundle OpenFact **5.6.1+**.

### 📚 Hiera

Hiera is the **hierarchical data lookup system** built into Puppet. It lets you separate your **data** (parameters, configuration values) from your **code** (classes, modules). Instead of hardcoding values in your manifests, you put them in YAML files organized in a hierarchy:

```text
data/
├── nodes/
│   └── webserver1.example.com.yaml    ← Most specific
├── os/
│   ├── RedHat.yaml
│   └── Debian.yaml
├── environment/
│   ├── production.yaml
│   └── staging.yaml
└── common.yaml                         ← Least specific (fallback)
```

Hiera searches from most-specific to least-specific, returning the first match. This means you can set defaults in `common.yaml` and override them per-node, per-OS, or per-environment. It's like CSS specificity, but for infrastructure. (We dive deep into Hiera in the [Hiera Deep-Dive](../hiera/README.md).)

---

## The Agent Run: Step by Step

Here's what happens during a typical agent run, from start to finish:

```text
Agent Node                                    Primary Server
──────────                                    ──────────────
1. Agent wakes up (timer or manual)
   │
2. Facter gathers system facts
   │
3. Agent sends facts ──────────────────────►  4. Server receives facts
                                              │
                                              5. Server compiles catalog
                                              │  (code + facts + Hiera data)
                                              │
4. Agent receives catalog ◄────────────────── 6. Server sends catalog
   │
5. Agent compares catalog to
   current system state
   │
6. Agent applies changes
   (creates files, installs packages,
    starts services, etc.)
   │
7. Agent sends report ────────────────────►   8. Server stores report
                                                 in PuppetDB
```

### Exit Codes Matter

When the agent finishes, it returns an exit code:

| Exit Code | Meaning |
|-----------|---------|
| `0` | No changes needed — system already matches desired state |
| `1` | Errors occurred during the run |
| `2` | Changes were successfully applied |
| `4` | Failures occurred (some resources failed) |
| `6` | Both changes and failures occurred |

> **Common gotcha:** Exit code **2** means "changes applied successfully." It's NOT an error! Many CI/CD systems treat non-zero exit codes as failures, so you may need to handle this explicitly. Puppet has been confusing automation engineers with this since approximately forever.

---

## Environments

Environments let you **isolate different versions of your Puppet code**. Each environment is simply a directory under the `environmentpath` (by default `/etc/puppetlabs/code/environments/`), and every environment is self-contained with its own manifests, modules, and data.

OpenVox ships with a single default environment: **`production`**. That's it — everything else is up to you and your workflow. There's no mandated set of environments; you create whatever makes sense for your organization.

When using **r10k** for code deployment (which most teams do), environments map **directly to Git branches** in your control repository. Create a branch, deploy with r10k, and a matching environment appears on the server. This means your environments are as dynamic as your Git workflow:

```text
Git Branch                              Puppet Environment
──────────                              ──────────────────
main               ─── r10k deploy ──►  production/
feature/add-nginx  ─── r10k deploy ──►  feature_add_nginx/
hotfix/ssl-cert    ─── r10k deploy ──►  hotfix_ssl_cert/
```

> **Note:** Puppet converts characters that aren't valid in environment names (like `/` and `-`) to underscores. The Git branch `feature/add-nginx` becomes the environment `feature_add_nginx`.

Each environment has its own:

- **Manifests** (`manifests/site.pp`)
- **Modules** (`modules/`)
- **Hiera data** (`data/`)
- **Hiera config** (`hiera.yaml`)
- **Environment config** (`environment.conf`)

The directory structure looks like this:

```text
/etc/puppetlabs/code/environments/
├── production/              ← The default (and often only permanent) environment
│   ├── manifests/
│   │   └── site.pp
│   ├── modules/             ← Managed by r10k (from Puppetfile)
│   ├── site-modules/        ← Your organization's custom modules
│   ├── data/
│   │   ├── common.yaml
│   │   └── nodes/
│   ├── hiera.yaml
│   ├── environment.conf
│   └── Puppetfile
├── feature_add_nginx/       ← Created automatically by r10k from a Git branch
│   └── ...
└── hotfix_ssl_cert/         ← Temporary — removed when the branch is merged/deleted
    └── ...
```

Agents are assigned to environments in one of three ways:

1. **`puppet.conf`** — set `environment = production` in the `[agent]` section (this is the default)
2. **External Node Classifier (ENC)** — a script or service that tells the server which environment a node belongs to
3. **Command line** — `puppet agent -t --environment feature_add_nginx` (great for testing a branch on a single node before merging)

---

## The Module System

Modules are how you **organize and share Puppet code**. A module is a directory with a specific structure:

```text
mymodule/
├── manifests/
│   ├── init.pp          ← Main class (class mymodule)
│   ├── config.pp        ← Configuration subclass
│   ├── install.pp       ← Package installation
│   └── service.pp       ← Service management
├── files/               ← Static files to distribute
├── templates/           ← ERB or EPP templates
├── lib/
│   ├── facter/          ← Custom facts
│   └── puppet/
│       ├── functions/   ← Custom functions
│       └── types/       ← Custom types and providers
├── data/                ← Module-level Hiera data
├── spec/                ← Tests (rspec-puppet)
├── examples/            ← Usage examples
└── metadata.json        ← Module metadata (name, version, deps)
```

The [Puppet Forge](https://forge.puppet.com/) hosts thousands of community modules. Because OpenVox is a drop-in replacement for Puppet, **all Forge modules work with OpenVox** without modification.

---

## How Data Flows

Here's a complete picture of how data flows through the system:

```text
┌─────────────┐          ┌──────────────┐
│  Git Repo   │──r10k───►│  Code Dir    │
│  (control   │  deploy  │  /etc/puppet │
│   repo)     │          │  labs/code/  │
└─────────────┘          └──────┬───────┘
                                │
┌─────────────┐          ┌──────┴───────┐          ┌──────────┐
│   Agent     │──facts──►│ PuppetServer │──store──►│ PuppetDB │
│   (node)    │          │  (compiler)  │          │ (Postgres│
│             │◄─catalog─│              │◄─query───│  backend)│
│             │          └──────────────┘          └──────────┘
│             │
│  ┌────────┐ │
│  │ Facter │ │  (gathers facts)
│  └────────┘ │
│  ┌────────┐ │
│  │ Report │─┼──report──► PuppetDB
│  └────────┘ │
└─────────────┘
```

---

## Glossary

| Term | Definition |
|------|-----------|
| **Agent** | The software running on managed nodes that applies catalogs |
| **Catalog** | A compiled document describing all resources and their desired state |
| **CA** | Certificate Authority — manages SSL certificates for mTLS |
| **Certname** | A node's unique identifier (usually its FQDN) |
| **ENC** | External Node Classifier — assigns classes/parameters to nodes |
| **Environment** | An isolated set of Puppet code (production, staging, etc.) |
| **Fact** | A piece of information about a node (OS, IP, RAM, etc.) |
| **Forge** | The Puppet Forge — community module repository |
| **Hiera** | Hierarchical data lookup system |
| **Idempotent** | Can be applied repeatedly with the same result |
| **Manifest** | A `.pp` file containing Puppet code |
| **Module** | A self-contained bundle of Puppet code |
| **mTLS** | Mutual TLS — both client and server verify each other's certificates |
| **Node** | A managed system (server, VM, container, etc.) |
| **OpenBolt** | OpenVox's name for Bolt; package `openbolt`, binary still `bolt` |
| **OpenFact** | OpenVox's name for Facter; binary still `facter` |
| **OpenVoxDB** | OpenVox's name for PuppetDB; packages `openvoxdb`, `openvoxdb-termini` |
| **PQL** | Puppet Query Language — SQL-like language for querying PuppetDB |
| **Primary Server** | The central PuppetServer that compiles catalogs |
| **Resource** | A single unit of configuration (file, package, service, etc.) |
| **r10k** | Tool for deploying Puppet code from Git |

---

*Next up: [The Puppet Language →](../language/README.md)*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
