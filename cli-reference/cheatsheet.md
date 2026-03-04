# Utility Commands Cheat Sheet

> *Copy, paste, conquer.*

[← Back to CLI Reference](README.md)

---

Quick reference for day-to-day operations:

```bash
# ─── Agent Operations ───
sudo puppet agent -t                    # Test run
sudo puppet agent -t --noop             # Dry run
sudo puppet agent --disable "reason"    # Lock agent
sudo puppet agent --enable              # Unlock agent

# ─── Local Apply ───
sudo puppet apply manifest.pp           # Apply a manifest
sudo puppet apply -e 'code'             # Apply inline code
puppet parser validate manifest.pp      # Check syntax

# ─── Inspection ───
puppet resource user                    # List all users
puppet resource package httpd           # Check package state
puppet resource service sshd            # Check service state
puppet describe file                    # Docs for file type

# ─── Configuration ───
puppet config print server              # Show server setting
puppet config print all                 # Show all settings
sudo puppet config set server host.com  # Set server

# ─── Facts ───
facter                                  # All facts
facter os.name                          # Specific fact
facter --json                           # JSON output

# ─── SSL ───
sudo puppet ssl show                    # Show certificate
sudo puppet ssl verify                  # Verify cert chain
sudo puppet ssl clean                   # Remove local certs

# ─── Modules ───
puppet module list                      # List installed
sudo puppet module install author-mod   # Install from Forge
puppet module search keyword            # Search Forge

# ─── Server / CA ───
sudo puppetserver ca list               # Pending CSRs
sudo puppetserver ca list --all         # All certs
sudo puppetserver ca sign --certname X  # Sign a CSR
sudo puppetserver ca revoke --certname X # Revoke a cert
sudo puppetserver ca clean --certname X # Remove a cert

# ─── Hiera Lookup ───
puppet lookup myclass::param            # Lookup a key
puppet lookup key --explain             # Show hierarchy
puppet lookup key --merge deep          # Deep merge

# ─── Code Deployment ───
sudo r10k deploy environment --verbose  # Deploy all envs
sudo r10k deploy environment prod -v    # Deploy one env

# ─── Orchestration (Bolt) ───
bolt command run 'cmd' --targets host   # Run command
bolt task run task --targets host       # Run task
bolt plan run plan --targets host       # Run plan
```

[← Back to CLI Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>