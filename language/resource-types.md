# Resource Types Reference

[← Back to Language Reference](README.md)

---


Here are the most commonly used built-in resource types. Think of these as your infrastructure vocabulary.

### `file` — Managing Files and Directories

```puppet
# Create a file with specific content
file { '/etc/banner.txt':
  ensure  => file,
  content => "Authorized access only.\n",
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
}

# Create a directory
file { '/opt/myapp':
  ensure => directory,
  owner  => 'appuser',
  group  => 'appgroup',
  mode   => '0755',
}

# Create a symlink
file { '/usr/local/bin/myapp':
  ensure => link,
  target => '/opt/myapp/bin/myapp',
}

# Copy a file from a module
file { '/etc/myapp/config.yml':
  ensure => file,
  source => 'puppet:///modules/myapp/config.yml',
  owner  => 'root',
  mode   => '0600',
}

# Use a template
file { '/etc/myapp/app.conf':
  ensure  => file,
  content => epp('myapp/app.conf.epp'),
}

# Recursively manage a directory
file { '/opt/myapp/conf.d':
  ensure  => directory,
  recurse => true,
  purge   => true,    # Remove files not managed by Puppet
  source  => 'puppet:///modules/myapp/conf.d',
}
```

**Key attributes:**

| Attribute | Description | Values |
|-----------|-------------|--------|
| `ensure` | What to create | `file`, `directory`, `link`, `absent` |
| `content` | File contents (mutually exclusive with `source`) | String |
| `source` | Copy from source (puppet:// URI, local path, or HTTP) | URI/path |
| `owner` | File owner | Username or UID |
| `group` | File group | Group name or GID |
| `mode` | Permissions | Octal string like `'0644'` |
| `recurse` | Manage directory contents recursively | `true`, `false` |
| `purge` | Remove unmanaged files in directory | `true`, `false` |
| `target` | Symlink target | Path |

### `package` — Installing Software

```puppet
# Ensure a package is installed (latest available at install time)
package { 'nginx':
  ensure => installed,
}

# Pin to a specific version
package { 'nodejs':
  ensure => '18.19.0-1.el9',
}

# Ensure latest version (upgrades on every run!)
package { 'security-updates':
  name   => 'openssl',
  ensure => latest,
}

# Remove a package
package { 'telnet':
  ensure => absent,
}

# Install multiple packages at once
$web_packages = ['httpd', 'mod_ssl', 'php', 'php-mysqlnd']
package { $web_packages:
  ensure => installed,
}
```

> **Warning:** Using `ensure => latest` means Puppet will upgrade the package on *every* run if a newer version is available. This can be dangerous in production! Prefer `installed` or a pinned version.

### `service` — Managing Services

```puppet
# Ensure a service is running and enabled at boot
service { 'httpd':
  ensure => running,
  enable => true,
}

# Stop and disable a service
service { 'cups':
  ensure => stopped,
  enable => false,
}

# Restart a service when its config file changes
service { 'nginx':
  ensure    => running,
  enable    => true,
  subscribe => File['/etc/nginx/nginx.conf'],
}
```

### `user` — Managing User Accounts

```puppet
# Create a user with full details
user { 'deploy':
  ensure     => present,
  uid        => 5000,
  gid        => 'deploy',
  home       => '/home/deploy',
  shell      => '/bin/bash',
  managehome => true,
  comment    => 'Deployment user',
}

# Remove a user
user { 'oldemployee':
  ensure => absent,
}
```

### `group` — Managing Groups

```puppet
group { 'deploy':
  ensure => present,
  gid    => 5000,
}
```

### `cron` — Managing Cron Jobs

```puppet
cron { 'db_backup':
  command => '/usr/local/bin/backup.sh',
  user    => 'root',
  hour    => 2,
  minute  => 30,
}
```

### `exec` — Running Commands (Use Sparingly!)

```puppet
# Only run if the target file doesn't exist
exec { 'initialize_database':
  command => '/usr/local/bin/db-init.sh',
  creates => '/var/lib/myapp/db.sqlite',
  user    => 'appuser',
}

# Only run when a condition is true
exec { 'rebuild_cache':
  command     => '/usr/local/bin/rebuild-cache.sh',
  onlyif      => 'test -f /var/lib/myapp/cache.stale',
  refreshonly => true,
}
```

> **Warning:** `exec` is the escape hatch of Puppet. It runs arbitrary commands, which makes it hard to guarantee idempotence. Always use `creates`, `onlyif`, `unless`, or `refreshonly` to prevent it from running on every Puppet run. If you find yourself writing lots of `exec` resources, you probably want a custom type or a different approach.

---

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
