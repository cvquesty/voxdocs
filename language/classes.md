# Classes

[← Back to Language Reference](README.md)

---

Classes are **named blocks of Puppet code** that can be declared (included) on a node. They're the primary way to organize your Puppet code.

## Defining a Class

```puppet
# modules/webserver/manifests/init.pp
class webserver (
  Integer        $port    = 80,
  Boolean        $ssl     = false,
  Array[String]  $modules = ['mod_ssl', 'mod_rewrite'],
) {
  package { 'httpd':
    ensure => installed,
  }

  file { '/etc/httpd/conf/httpd.conf':
    ensure  => file,
    content => epp('webserver/httpd.conf.epp', {
      'port'    => $port,
      'ssl'     => $ssl,
      'modules' => $modules,
    }),
    notify  => Service['httpd'],
  }

  service { 'httpd':
    ensure => running,
    enable => true,
  }
}
```

## Declaring (Using) a Class

```puppet
# Method 1: include (uses Hiera for parameters)
include webserver

# Method 2: resource-like declaration (sets parameters explicitly)
class { 'webserver':
  port => 8080,
  ssl  => true,
}
```

> **Best practice:** Use `include` with Hiera data for parameters. Resource-like declarations (`class { ... }`) can only be used once per class and are less flexible.

## The `contain` and `require` Functions

```puppet
class myapp {
  contain myapp::install   # Include + contain within this class's scope
  contain myapp::config
  contain myapp::service

  Class['myapp::install']
    -> Class['myapp::config']
    ~> Class['myapp::service']
}
```

---

[← Back to Language Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
