# puppet config

> *Think of it as the `git config` of the Puppet world.*

[← Back to CLI Reference](README.md)

---

Read and modify Puppet configuration settings.

```console
$ puppet help config

USAGE: puppet config <action> [--section SECTION_NAME]

This subcommand can inspect and modify settings from OpenVox's
'puppet.conf' configuration file.

OPTIONS:
  --render-as FORMAT             - The rendering format to use.
  --verbose                      - Whether to log verbosely.
  --debug                        - Whether to log debug information.
  --section SECTION_NAME         - The section of the configuration file to
                                   interact with.

ACTIONS:
  delete    Delete an OpenVox setting.
  print     Examine OpenVox's current settings.
  set       Set OpenVox's settings.
```

## Common Usage Patterns

```bash
# Print ALL settings (there are hundreds)
puppet config print all

# Print a specific setting
puppet config print server
puppet config print certname
puppet config print runinterval
puppet config print modulepath
puppet config print environmentpath

# Set the server hostname
sudo puppet config set server openvox.example.com --section agent

# Set the run interval to 1 hour
sudo puppet config set runinterval 3600 --section agent

# Set the environment
sudo puppet config set environment production --section agent

# Delete a setting (revert to default)
sudo puppet config delete server --section agent
```

[← Back to CLI Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
