# Defined Types

[← Back to Language Reference](README.md)

---

Defined types are like **reusable resource templates**. Unlike classes (which can only be declared once), defined types can be instantiated multiple times with different parameters:

```puppet
# modules/vhost/manifests/init.pp
define vhost (
  String  $docroot,
  Integer $port      = 80,
  String  $server_name = $title,
  Boolean $ssl       = false,
) {
  file { "/etc/httpd/conf.d/${title}.conf":
    ensure  => file,
    content => epp('vhost/vhost.conf.epp', {
      'server_name' => $server_name,
      'docroot'     => $docroot,
      'port'        => $port,
      'ssl'         => $ssl,
    }),
    notify  => Service['httpd'],
  }

  file { $docroot:
    ensure => directory,
    owner  => 'apache',
    group  => 'apache',
  }
}
```

Using it:

```puppet
vhost { 'blog':
  docroot     => '/var/www/blog',
  server_name => 'blog.example.com',
}

vhost { 'api':
  docroot     => '/var/www/api',
  server_name => 'api.example.com',
  port        => 8080,
  ssl         => true,
}
```

---

[← Back to Language Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
