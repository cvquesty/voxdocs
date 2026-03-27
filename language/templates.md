# Templates

[← Back to Language Reference](README.md)

---


Templates let you generate file content dynamically. Puppet supports two template languages:

### EPP (Embedded Puppet) — Preferred

```puppet
# In your manifest:
file { '/etc/myapp.conf':
  content => epp('myapp/config.epp', {
    'port'     => $port,
    'workers'  => $facts['processors']['count'],
    'hostname' => $facts['networking']['fqdn'],
  }),
}
```

```epp
<%# myapp/templates/config.epp %>
<%- | Integer $port, Integer $workers, String $hostname | -%>
# Managed by OpenVox — do not edit manually
# Generated for <%= $hostname %>

listen_port = <%= $port %>
worker_processes = <%= $workers %>
<%- if $workers > 4 { -%>
# High-performance mode enabled
thread_pool_size = <%= $workers * 2 %>
<%- } -%>
```

### ERB (Embedded Ruby) — Legacy

```erb
<%# myapp/templates/config.erb %>
# Managed by OpenVox
listen_port = <%= @port %>
worker_processes = <%= @processors['count'] %>
```

> **Note:** EPP is the preferred template format in modern Puppet/OpenVox. ERB still works but EPP has cleaner syntax and better type safety. New code should always use EPP.

---

[← Back to Language Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
