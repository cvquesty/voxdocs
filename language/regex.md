# Regular Expressions

[← Back to Language Reference](README.md)

---


Puppet uses Ruby-style regular expressions:

```puppet
# Match operator
if $facts['networking']['fqdn'] =~ /^web\d+\.example\.com$/ {
  include webserver
}

# Not-match operator
if $facts['os']['name'] !~ /^(RedHat|CentOS|Rocky)$/ {
  notify { 'This is not a RHEL-family system': }
}

# Capture groups
if $facts['networking']['fqdn'] =~ /^(\w+)\.(\w+)\.example\.com$/ {
  $role = $1       # e.g., 'web'
  $datacenter = $2 # e.g., 'us-east'
}

# In case statements
case $facts['os']['name'] {
  /^(RedHat|CentOS|Rocky|Alma)/: { $family = 'rhel' }
  /^(Debian|Ubuntu)/:             { $family = 'debian' }
}
```

---

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
