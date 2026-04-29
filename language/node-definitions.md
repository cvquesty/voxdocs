# Node Definitions

[← Back to Language Reference](README.md)

---

Node definitions let you assign code to specific nodes in your `site.pp`:

```puppet
# manifests/site.pp

# Default for all nodes
node default {
  include base_profile
}

# Specific node
node 'webserver1.example.com' {
  include base_profile
  include webserver
  include monitoring
}

# Regex matching
node /^db\d+\.example\.com$/ {
  include base_profile
  include database
}
```

> **Pro tip:** In modern Puppet/OpenVox, many teams use **roles and profiles** with Hiera instead of node definitions. This is more flexible and easier to manage at scale. See the [Module Development Guide](../module-development/README.md) for the roles/profiles pattern.

---

[← Back to Language Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
