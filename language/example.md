# Putting It All Together

[← Back to Language Reference](README.md)

---


Here's a realistic module that installs and configures NTP:

```puppet
# modules/ntp/manifests/init.pp
class ntp (
  Array[String] $servers  = ['0.pool.ntp.org', '1.pool.ntp.org'],
  Boolean       $restrict = true,
  String        $timezone = 'UTC',
) {
  # Determine package name based on OS
  $ntp_package = $facts['os']['family'] ? {
    'RedHat' => 'chrony',
    'Debian' => 'chrony',
    default  => 'ntp',
  }

  $ntp_service = $facts['os']['family'] ? {
    'RedHat' => 'chronyd',
    'Debian' => 'chrony',
    default  => 'ntpd',
  }

  # Install → Configure → Service
  package { $ntp_package:
    ensure => installed,
  }

  file { '/etc/chrony.conf':
    ensure  => file,
    content => epp('ntp/chrony.conf.epp', {
      'servers'  => $servers,
      'restrict' => $restrict,
    }),
    require => Package[$ntp_package],
    notify  => Service[$ntp_service],
  }

  service { $ntp_service:
    ensure => running,
    enable => true,
  }

  # Set the timezone
  exec { 'set_timezone':
    command => "/usr/bin/timedatectl set-timezone ${timezone}",
    unless  => "/usr/bin/timedatectl status | grep 'Time zone: ${timezone}'",
  }
}
```


<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
