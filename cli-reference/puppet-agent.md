# puppet agent

> *The command you'll type more than any other in your OpenVox career.*

[← Back to CLI Reference](README.md)

---

The agent daemon — connects to the Primary Server, retrieves a catalog, and applies it.

```console
$ puppet help agent

puppet-agent(8) -- The puppet agent daemon provided by OpenVox
========

SYNOPSIS
--------
Retrieves the client configuration from the OpenVox server and applies it to
the local host.

This service may be run as a daemon, run periodically using cron (or something
similar), or run interactively for testing purposes.

USAGE
-----
puppet agent [--certname <NAME>] [-D|--daemonize|--no-daemonize]
  [-d|--debug] [--detailed-exitcodes] [--digest <DIGEST>] [--disable [MESSAGE]] [--enable]
  [--fingerprint] [-h|--help] [-l|--logdest syslog|eventlog|<ABS FILEPATH>|console]
  [--serverport <PORT>] [--noop] [-o|--onetime] [--sourceaddress <IP_ADDRESS>] [-t|--test]
  [-v|--verbose] [-V|--version] [-w|--waitforcert <SECONDS>]

DESCRIPTION
-----------
This is the main puppet client. Its job is to retrieve the local
machine's configuration from a remote server and apply it. In order to
successfully communicate with the remote server, the client must have a
certificate signed by a certificate authority that the server trusts;
the recommended method for this, at the moment, is to run a certificate
authority as part of the puppet server (which is the default). The
client will connect and request a signed certificate, and will continue
connecting until it receives one.

Once the client has a signed certificate, it will retrieve its
configuration and apply it.

OPTIONS
-------
Note that any Puppet setting that's valid in the configuration file is also a
valid long argument. For example, 'server' is a valid setting, so you can
specify '--server <servername>' as an argument. Boolean settings accept a '--no-'
prefix to turn off a behavior.

* --certname:        Set the certname (unique ID) of the client.
* --daemonize:       Send the process into the background (default).
* --no-daemonize:    Do not send the process into the background.
* --debug:           Enable full debugging.
* --detailed-exitcodes: Provide extra information via exit codes (see below).
* --digest:          Change the certificate fingerprint digest algorithm (default: SHA256).
* --disable [MSG]:   Disable puppet agent runs (optional reason message).
* --enable:          Re-enable puppet agent runs.
* --evaltrace:       Log each resource as it is evaluated.
* --fingerprint:     Display the current certificate fingerprint and exit.
* --help:            Print this help message.
* --job-id:          Attach a job ID to the catalog request and report (--onetime only).
* --logdest:         Where to send log messages (syslog, eventlog, console, or file path).
* --noop:            Dry-run mode — show what would change, change nothing.
* --onetime:         Run the configuration once and exit.
* --serverport:      Port to contact the OpenVox server on (default: 8140).
* --sourceaddress:   Set the source IP for outbound connections.
* --test:            Run in test mode (--onetime --verbose --no-daemonize
                     --no-usecacheonfailure --detailed-exitcodes --no-splay --show_diff).
* --trace:           Print stack traces on errors.
* --verbose:         Turn on verbose reporting.
* --version:         Print the puppet version number and exit.
* --waitforcert:     Seconds to wait for cert signing (default: 120, 0 to disable).
* --write_catalog_summary: Save resource/class lists to state directory after compilation.

SIGNALS
-------
  SIGHUP:   Restart the puppet agent daemon.
  SIGINT:   Shut down the puppet agent daemon.
  SIGTERM:  Shut down the puppet agent daemon.
  SIGUSR1:  Immediately retrieve and apply configurations.
  SIGUSR2:  Close and reopen log file descriptors (for logrotate).

COPYRIGHT
---------
Copyright (c) 2011 Puppet Inc.
Copyright (c) 2024 Vox Pupuli
Licensed under the Apache 2.0 License
```

## Common Usage Patterns

```bash
# Run once in test mode — the command you'll type 10,000 times
sudo puppet agent -t

# Dry run — see what WOULD change without changing anything
sudo puppet agent -t --noop

# Run against a specific server
sudo puppet agent -t --server=openvox.example.com

# Run in a specific environment (great for testing feature branches)
sudo puppet agent -t --environment=staging

# Disable the agent with a reason (shows up in reports)
sudo puppet agent --disable "Maintenance window - admin 2026-03-04"

# Re-enable the agent
sudo puppet agent --enable

# Check the certificate fingerprint
sudo puppet agent --fingerprint

# Run with full debug output (prepare for a LOT of text)
sudo puppet agent -t --debug

# Run with resource-level timing (find slow resources)
sudo puppet agent -t --evaltrace

# Only apply resources tagged with a specific class
sudo puppet agent -t --tags 'profile::webserver'
```

## Exit Codes

With `--detailed-exitcodes` or `--test`:

| Code | Meaning | CI/CD Treatment |
|------|---------|-----------------|
| `0` | No changes — system already in desired state | ✅ Success |
| `1` | An error occurred or run was blocked | ❌ Failure |
| `2` | Changes were applied successfully | ✅ Success (not an error!) |
| `4` | Some resources failed | ❌ Failure |
| `6` | Changes applied AND failures occurred | ❌ Failure |

> **⚠️ Classic gotcha:** Exit code **2** means "I made changes and they all worked." It is NOT an error! Many CI/CD systems treat any non-zero exit code as failure. See [Troubleshooting](../troubleshooting/README.md) for the fix.

[← Back to CLI Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
