# puppetserver

> *The JVM-based brains of the operation — and your Certificate Authority.*

[← Back to CLI Reference](README.md)

---

The PuppetServer binary manages the JVM-based server process and the built-in Certificate Authority. This runs on the Primary Server only.

## `puppetserver --help`

```
$ puppetserver --help

usage: puppetserver ([--help] | [--version]) <command> [<args>]

The most commonly used puppetserver commands are:
   ca
   foreground
   gem
   irb
   prune
   reload
   ruby
   start
   stop

See 'puppetserver <command> -h' for more information on a specific command.
```

## `puppetserver version`

```
$ puppetserver version
puppetserver version: 8.12.1
```

## `puppetserver ca`

The built-in Certificate Authority management tool. This is how you manage the PKI infrastructure that secures all agent↔server communication.

```
$ puppetserver ca --help

Usage: puppetserver ca <action> [options]

Manage the Private Key Infrastructure for
Puppet Server's built-in Certificate Authority

Available Actions:

  Certificate Actions (requires a running Puppet Server):

    clean       Revoke cert(s) and remove related files from CA
    generate    Generate a new certificate signed by the CA
    list        List certificates and CSRs
    revoke      Revoke certificate(s)
    sign        Sign certificate request(s)

  Administrative Actions (requires Puppet Server to be stopped):

    delete      Delete signed certificate(s) from disk
    import      Import an external CA chain and generate server PKI
    setup       Setup a self-signed CA chain for Puppet Server
    enable      Setup infrastructure CRL based on a node inventory.
    migrate     Migrate the existing CA directory
    prune       Prune the local CRL on disk to remove certificate entries

General Options:
        --help       Display this general help output
        --version    Display the version
        --verbose    Display low-level information
```

## Common Usage Patterns

```bash
# List all certificate signing requests (pending + signed)
sudo puppetserver ca list --all

# List only pending (unsigned) requests
sudo puppetserver ca list

# Sign a specific certificate request
sudo puppetserver ca sign --certname agent1.example.com

# Sign ALL pending requests
sudo puppetserver ca sign --all

# Revoke a certificate (compromised or decommissioned node)
sudo puppetserver ca revoke --certname old-server.example.com

# Clean a certificate (remove from CA entirely)
sudo puppetserver ca clean --certname old-server.example.com

# Generate a certificate for a specific node
sudo puppetserver ca generate --certname new-service.example.com
```

## Live Example

From `openvox.example.com`:

```
$ sudo puppetserver ca list --all

Signed Certificates:
    openvox.example.com       (SHA256)  F9:70:1B:30:19:46:10:5D:7A:19:41:94:8D:40:92:34:...
        alt names: ["DNS:puppet", "DNS:openvox.example.com"]
        authorization extensions: [pp_cli_auth: true]
    agent1.example.com        (SHA256)  94:2C:B9:EA:C4:16:98:0A:52:D2:71:BA:3E:BC:76:56:...
        alt names: ["DNS:agent1.example.com"]
    agent2.example.com        (SHA256)  17:26:8C:66:4D:B0:43:F4:96:FE:D0:D4:72:FB:C3:37:...
        alt names: ["DNS:agent2.example.com"]
```

> **Pro tip:** Notice the server cert has `alt names` including `puppet` — this is the `dns_alt_names` setting. The `pp_cli_auth: true` extension means this cert can be used for CLI-based CA operations.

[← Back to CLI Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>