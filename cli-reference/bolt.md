# bolt (OpenBolt)

> *Ansible's opinionated cousin who insists on doing things the Puppet way.*

[← Back to CLI Reference](README.md)

---

Bolt — now **OpenBolt** in the OpenVox ecosystem — is an agentless orchestration tool. It connects to targets via SSH (or WinRM for Windows) and runs commands, scripts, tasks, and plans without requiring a Puppet agent.

## `bolt --help`

```
$ bolt --help

Name
    OpenBolt

Usage
    bolt <subcommand> [action] [options]

Description
    OpenBolt is an orchestration tool that automates the manual work it takes to
    maintain your infrastructure.

Subcommands
    apply             Apply Puppet manifest code
    command           Run a command remotely
    file              Copy files between the controller and targets
    group             Show the list of groups in the inventory
    guide             View guides for Bolt concepts and features
    inventory         Show the list of targets an action would run on
    module            Manage Bolt project modules
    lookup            Look up a value with Hiera
    plan              Convert, create, show, and run Bolt plans
    plugin            Show available plugins
    policy            Apply, create, and show policies
    project           Create and migrate Bolt projects
    script            Upload a local script and run it remotely
    secret            Create encryption keys and encrypt and decrypt values
    task              Show and run Bolt tasks

Guides
    For a list of guides on Bolt's concepts and features, run 'bolt guide'.
    Find Bolt's documentation at https://bolt.guide.

Global options
    -h, --help                       Display help.
        --version                    Display the version.
        --log-level LEVEL            Set the log level for the console. Available options are
                                     trace, debug, info, warn, error, fatal.
        --clear-cache                Clear plugin, plan, and task caches before executing.
```

## `bolt --version`

```
$ bolt --version
5.4.0
```

> **Note:** The community fork is called **OpenBolt** and identifies itself as such in the `--help` output. It includes additional subcommands like `policy` and `plugin` not present in older Puppet Bolt versions.

## Common Usage Patterns

```bash
# Run a command on remote targets
bolt command run 'uptime' --targets webservers

# Run a command on a single host
bolt command run 'df -h' --targets agent1.example.com

# Upload a file
bolt file upload /local/path/file.conf /remote/path/file.conf --targets all

# Download a file
bolt file download /var/log/messages ./logs/ --targets agent1.example.com

# Run a script
bolt script run ./deploy.sh --targets webservers

# Run a Puppet task
bolt task run package action=install name=httpd --targets webservers

# Run a Bolt plan
bolt plan run myapp::deploy version=2.0 --targets webservers

# Apply a Puppet manifest (agentless!)
bolt apply manifest.pp --targets agent1.example.com

# Apply inline Puppet code
bolt apply -e 'package { "vim": ensure => installed }' --targets all

# Show inventory
bolt inventory show --targets all

# Lookup Hiera data
bolt lookup myapp::db_password --targets dbserver.example.com

# Use PuppetDB for target discovery
bolt command run 'hostname' --query 'nodes[certname] { facts.os.name = "Rocky" }'
```

## Connection Options

```bash
# SSH with specific user and key
bolt command run 'id' --targets host.example.com \
  --user deploy --private-key ~/.ssh/id_ed25519

# WinRM for Windows targets
bolt command run 'Get-Service' --targets winhost.example.com \
  --transport winrm --user Administrator --password

# Using an inventory file
bolt command run 'uptime' --targets all -i inventory.yaml

# Limit concurrency
bolt command run 'apt update' --targets all --concurrency 10

# Output as JSON
bolt command run 'hostname -f' --targets all --format json
```

[← Back to CLI Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>