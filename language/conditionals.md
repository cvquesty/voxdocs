# Conditionals

[← Back to Language Reference](README.md)

---


### `if` / `elsif` / `else`

```puppet
if $facts['os']['family'] == 'RedHat' {
  $web_package = 'httpd'
  $web_service = 'httpd'
} elsif $facts['os']['family'] == 'Debian' {
  $web_package = 'apache2'
  $web_service = 'apache2'
} else {
  fail("Unsupported OS family: ${facts['os']['family']}")
}

package { $web_package: ensure => installed }
service { $web_service: ensure => running }
```

### `case`

```puppet
case $facts['os']['name'] {
  'RedHat', 'CentOS', 'Rocky', 'AlmaLinux': {
    $package_manager = 'yum'
  }
  'Debian', 'Ubuntu': {
    $package_manager = 'apt'
  }
  /^(SLES|SUSE)/: {  # Regex matching!
    $package_manager = 'zypper'
  }
  default: {
    fail("Unknown OS: ${facts['os']['name']}")
  }
}
```

### Selector (Ternary-style)

```puppet
$web_package = $facts['os']['family'] ? {
  'RedHat' => 'httpd',
  'Debian' => 'apache2',
  default  => 'httpd',
}
```

---

[← Back to Language Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
