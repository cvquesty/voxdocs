# puppet query (PuppetDB / PQL)

> *SQL for your infrastructure — once you learn it, you'll wonder how you ever lived without it.*

[← Back to CLI Reference](README.md)

---

Query PuppetDB using PQL (Puppet Query Language).

```bash
# Find all nodes
puppet query 'nodes {}'

# Find nodes by OS
puppet query 'nodes[certname] { facts.os.name = "RedHat" }'

# Find nodes with a specific package
puppet query 'resources[certname] { type = "Package" and title = "httpd" }'

# Find facts for a node
puppet query 'facts { certname = "agent1.example.com" }'

# Find nodes that changed on last run
puppet query 'nodes[certname] { latest_report_status = "changed" }'

# Find nodes that haven't checked in for 2 hours
puppet query 'nodes[certname] { report_timestamp < "2 hours ago" }'

# Count nodes by OS family
puppet query 'facts[value, count()] { name = "os.family" group by value }'

# Find all nodes with a specific class applied
puppet query 'resources[certname] { type = "Class" and title = "Profile::Base" }'

# Export PuppetDB data
puppet db export backup.tgz

# Import PuppetDB data
puppet db import backup.tgz
```

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>