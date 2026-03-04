# Iteration

[← Back to Language Reference](README.md)

---


Puppet supports several iteration methods (added in Puppet 4+):

### `each`

```puppet
['httpd', 'php', 'mysql'].each |$pkg| {
  package { $pkg:
    ensure => installed,
  }
}

# With index
['web', 'app', 'db'].each |$index, $role| {
  notify { "Server ${index}: ${role}": }
}
```

### `map`

```puppet
$ports = [80, 443, 8080]
$firewall_rules = $ports.map |$port| {
  "allow_${port}"
}
# Result: ['allow_80', 'allow_443', 'allow_8080']
```

### `filter`

```puppet
$all_packages = ['httpd', 'telnet', 'php', 'ftp']
$secure_packages = $all_packages.filter |$pkg| {
  $pkg !~ /^(telnet|ftp)$/
}
# Result: ['httpd', 'php']
```

### `reduce`

```puppet
$numbers = [1, 2, 3, 4, 5]
$sum = $numbers.reduce(0) |$memo, $n| {
  $memo + $n
}
# Result: 15
```

---

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
