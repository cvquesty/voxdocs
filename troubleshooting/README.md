# 🔧 Troubleshooting & FAQ

> *Because stuff breaks. It's fine. We'll figure it out together.*

---

## Common Issues

### 🔴 "Could not request certificate" / SSL Errors

**Symptoms:** Agent can't connect to the server. SSL handshake failures. Certificate errors.

**Common fixes:**

```bash
# 1. Check if the server is reachable
curl -k https://your-server:8140

# 2. Clean local SSL and re-register
sudo puppet ssl clean
sudo puppet agent -t

# 3. On the server, check for stale certs
sudo puppetserver ca list --all
sudo puppetserver ca clean --certname problematic-node.example.com

# 4. Verify time sync (SSL is time-sensitive!)
date                    # On both server and agent
chronyc tracking        # Check NTP status
```

> **Pro tip:** Certificate issues are almost always caused by one of: wrong server hostname, DNS resolution failure, time skew, or stale certificates. Check those four things first and you'll solve 90% of SSL problems.

---

### 🔴 "Could not find class" / Compilation Errors

**Symptoms:** `Error: Could not find class 'mymodule::myclass'` during agent run.

**Checklist:**

1. **Is the module installed?**
   ```bash
   puppet module list | grep mymodule
   ```

2. **Is the file in the right place?** Class `mymodule::config` must be in `manifests/config.pp`:
   ```bash
   ls /etc/puppetlabs/code/environments/production/modules/mymodule/manifests/
   ```

3. **Has r10k been deployed recently?**
   ```bash
   sudo r10k deploy environment production --modules --verbose
   ```

4. **Is the environment correct?**
   ```bash
   puppet config print environment
   ```

5. **Is the environment cache stale?**
   ```bash
   sudo systemctl restart puppetserver  # Nuclear option
   ```

---

### 🔴 Exit Code 2 — "But It Says It Changed Things!"

**The situation:** Your CI/CD pipeline reports failure, but the Puppet run actually *succeeded*.

**The explanation:** Puppet exit code 2 means "I made changes successfully." It's not an error! When using `puppet agent -t` (which enables `--detailed-exitcodes`), the exit codes are:

| Code | Meaning |
|------|---------|
| 0 | No changes needed |
| 1 | Error occurred |
| **2** | **Changes applied successfully** ✅ |
| 4 | Failures occurred |
| 6 | Changes AND failures |

**Fix for CI/CD:**

```bash
# In your pipeline script:
puppet agent -t
EXIT_CODE=$?

if [ $EXIT_CODE -eq 0 ] || [ $EXIT_CODE -eq 2 ]; then
    echo "Puppet run successful"
    exit 0
else
    echo "Puppet run failed with exit code $EXIT_CODE"
    exit 1
fi
```

---

### 🔴 "Could not retrieve catalog" / Server Communication Failures

**Symptoms:** Agent can connect but can't get a catalog.

```bash
# Check PuppetServer logs
sudo journalctl -u puppetserver --since "10 minutes ago"

# Check for compilation errors (the most common cause)
sudo tail -f /var/log/puppetlabs/puppetserver/puppetserver.log

# Try a manual compilation on the server
sudo puppet catalog compile your-node.example.com --environment production
```

**Common causes:**
- Syntax error in a manifest (run `puppet parser validate` on your `.pp` files)
- Missing module dependency
- Hiera data error (YAML syntax)
- PuppetServer out of memory (check JVM heap)

---

### 🔴 PuppetServer Won't Start / OOM Kills

**Symptoms:** PuppetServer crashes or never finishes starting.

```bash
# Check for OOM kills
dmesg | grep -i "killed process"
journalctl -u puppetserver | grep -i "error\|out of memory\|heap"

# Check JVM heap settings
grep JAVA_ARGS /etc/sysconfig/puppetserver

# Reduce JRuby instances if low on memory
# Edit /etc/puppetlabs/puppetserver/conf.d/puppetserver.conf
# Set max-active-instances to 1 or 2
```

**Quick fix:** If you're on a small server (< 4GB RAM):

```bash
# Set a conservative heap
sudo sed -i 's/JAVA_ARGS=.*/JAVA_ARGS="-Xms1g -Xmx1g"/' /etc/sysconfig/puppetserver
sudo systemctl restart puppetserver
```

---

### 🔴 PuppetDB Connection Issues

**Symptoms:** `Error: Could not retrieve facts` or PuppetDB queries fail.

```bash
# Check PuppetDB service
sudo systemctl status puppetdb

# Check PuppetDB logs
sudo journalctl -u puppetdb --since "10 minutes ago"

# Verify the connection from the server
curl -s --cert /etc/puppetlabs/puppet/ssl/certs/$(hostname -f).pem \
     --key /etc/puppetlabs/puppet/ssl/private_keys/$(hostname -f).pem \
     --cacert /etc/puppetlabs/puppet/ssl/certs/ca.pem \
     https://localhost:8081/status/v1/services | python3 -m json.tool

# Check PostgreSQL
sudo systemctl status postgresql
sudo -u postgres psql puppetdb -c "SELECT 1;"
```

---

### 🔴 r10k Deployment Failures

**Symptoms:** `r10k deploy` fails or hangs.

```bash
# Common fix: check Git connectivity
ssh -T git@github.com  # Should say "Hi username!"

# Clear the cache and retry
sudo rm -rf /opt/puppetlabs/puppet/cache/r10k/
sudo r10k deploy environment --verbose --trace

# Check for Puppetfile syntax errors
cd /etc/puppetlabs/code/environments/production
r10k puppetfile check

# If using sudo, make sure the deploy key is accessible
sudo -u puppet ssh -T git@github.com
```

---

### 🟡 Agent Runs Are Slow

**Diagnosis:**

```bash
# Time a Puppet run with profiling
sudo puppet agent -t --profile

# Check fact gathering time
time facter --timing

# Check for slow exec resources (the usual suspect)
sudo puppet agent -t 2>&1 | grep -i "exec\|executed"
```

**Common speed-ups:**
- Remove unnecessary `exec` resources
- Use `facter --no-ruby` to skip slow Ruby facts
- Set `environment_timeout = unlimited` on the server
- Increase PuppetServer JRuby instances
- Use `splay = true` to spread agent check-ins

---

### 🟡 "Duplicate Declaration" Errors

**Symptoms:** `Error: Duplicate declaration: Type[title] is already declared`

This means you're trying to manage the same resource from two different places. Solutions:

1. **Use `ensure_packages()` from stdlib** instead of raw `package` declarations:
   ```puppet
   # Instead of:
   package { 'vim': ensure => installed }  # Fails if declared elsewhere

   # Use:
   ensure_packages(['vim'], { ensure => installed })  # Safe to call multiple times
   ```

2. **Use virtual resources** for shared packages:
   ```puppet
   @package { 'common_packages':
     name   => ['vim', 'curl', 'wget'],
     ensure => installed,
   }

   # Then realize where needed:
   realize Package['common_packages']
   ```

---

## FAQ

### Q: What's the difference between OpenVox and Puppet?

**A:** OpenVox is a **community-maintained fork** of Puppet, created by the Vox Pupuli community in 2025. It's functionally identical to Puppet 8.x — same language, same modules, same config files, same binaries. The difference is governance: OpenVox is 100% open source with community stewardship, while Puppet (Perforce) has moved toward proprietary models.

### Q: Can I use Puppet Forge modules with OpenVox?

**A:** Yes! All Puppet Forge modules work with OpenVox without any modifications. The underlying technology is identical.

### Q: Should I migrate from Puppet to OpenVox?

**A:** If you're running open-source Puppet, OpenVox is a drop-in replacement. Install `openvox-agent` instead of `puppet-agent`, and everything works. Your manifests, modules, Hiera data, and `puppet.conf` don't need to change.

### Q: Can OpenVox and Puppet agents coexist?

**A:** They share the same paths (`/etc/puppetlabs/`, `/opt/puppetlabs/`), so you shouldn't install both on the same machine. Choose one. Migration is straightforward — swap the packages.

### Q: How do I handle sensitive data?

**A:** Use **hiera-eyaml** for encrypted values in Hiera, and the `Sensitive` data type in Puppet code. See the [Hiera Deep-Dive](../hiera/README.md#encrypted-data-with-eyaml) for details.

### Q: What's the best way to test changes before production?

**A:** Use a **multi-environment workflow**:
1. Create a feature branch in your control repo
2. Deploy with r10k (creates a matching Puppet environment)
3. Test on a staging node: `puppet agent -t --environment your_branch`
4. Merge to production when satisfied
5. Deploy production with r10k

### Q: How often should agents run?

**A:** The default is every 30 minutes, which works well for most environments. For compliance-heavy environments, you might decrease to 15 minutes. For large fleets (1000+ nodes), you might increase to 1 hour with splay enabled to spread the load.

### Q: I installed openvox-agent 8.26.2 but `puppet --version` says `8.26.1`. Did the upgrade fail?

**A:** No — your agent is at the correct version. This is a known cosmetic bug in OpenVox 8.26.2: the version string returned by `--version` is wrong, but the agent itself is at 8.26.2. Tracked at [OpenVoxProject/openvox#415](https://github.com/OpenVoxProject/openvox/issues/415). You can verify with `rpm -q openvox-agent` (RHEL) or `dpkg -l openvox-agent` (Debian) — the package version is reported correctly there.

### Q: Where do I get help?

**A:** See the [Community & Contributing](../community/README.md) guide! The Puppet community Slack, Vox Pupuli mailing lists, and GitHub issues are all great places to ask questions.

---

## Diagnostic Commands Cheat Sheet

```bash
# ─── System Health ───
sudo systemctl status puppetserver puppetdb puppet
sudo journalctl -u puppetserver --since "1 hour ago"

# ─── Agent Diagnostics ───
puppet config print all                    # All settings
puppet config print server                 # Which server?
puppet config print certname               # What's my name?
puppet config print environment            # Which environment?
sudo puppet agent -t --noop               # What WOULD change?
sudo puppet agent -t --debug              # Maximum verbosity

# ─── Server Diagnostics ───
sudo puppet catalog compile $(hostname -f) # Test compilation
puppet parser validate manifest.pp         # Check syntax
sudo puppetserver ca list --all            # Certificate status

# ─── Hiera ───
puppet lookup myclass::param --explain     # Where's this value from?

# ─── Facts ───
facter --timing                            # What's slow?
facter os.name networking.ip               # Quick fact check

# ─── Code ───
r10k deploy display                        # What's deployed?
r10k puppetfile check                      # Puppetfile syntax OK?
```

---

*Next up: [Community & Contributing →](../community/README.md)*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
