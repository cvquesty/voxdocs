# Relationships & Ordering

[← Back to Language Reference](README.md)

---

By default, Puppet applies resources in a **non-deterministic order**. To enforce ordering, use relationships:

## Arrow Notation

```puppet
# Ordering: install package BEFORE starting service
Package['httpd'] -> Service['httpd']

# Notification: restart service WHEN config changes
File['/etc/httpd/conf/httpd.conf'] ~> Service['httpd']
```

| Arrow | Meaning |
|-------|---------|
| `->` | "before" (ordering only) |
| `~>` | "notify" (ordering + trigger refresh) |
| `<-` | "require" (reverse ordering) |
| `<~` | "subscribe" (reverse notify) |

## Metaparameters

```puppet
service { 'httpd':
  ensure    => running,
  require   => Package['httpd'],        # Don't start until package exists
  subscribe => File['/etc/httpd/conf/httpd.conf'],  # Restart when config changes
}

file { '/etc/httpd/conf/httpd.conf':
  ensure => file,
  notify => Service['httpd'],           # Same as subscribe, other direction
}

package { 'httpd':
  ensure => installed,
  before => Service['httpd'],           # Same as require, other direction
}
```

## Chaining

```puppet
# Common pattern: install → configure → service
package { 'nginx': ensure => installed }
-> file { '/etc/nginx/nginx.conf':
  ensure => file,
  source => 'puppet:///modules/nginx/nginx.conf',
}
~> service { 'nginx':
  ensure => running,
  enable => true,
}
```

---

[← Back to Language Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
