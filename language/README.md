# 📝 The Puppet Language Reference

> *"It's not programming, it's declaring your intentions to the universe. The universe just happens to be made of servers."*

---

## Overview

The Puppet language (also called the Puppet DSL) is a **declarative, domain-specific language** designed for one thing: describing the desired state of your infrastructure. You don't tell Puppet *how* to do something — you tell it *what you want*, and it figures out the rest.

If you're coming from shell scripting or Python, this requires a small shift in thinking. Instead of writing:

```bash
# Imperative (shell): HOW to do it
if ! rpm -q httpd; then
    yum install -y httpd
fi
systemctl enable httpd
systemctl start httpd
```

You write:

```puppet
# Declarative (Puppet): WHAT you want
package { 'httpd':
  ensure => installed,
}

service { 'httpd':
  ensure => running,
  enable => true,
}
```

Same result, but the Puppet version is idempotent, cross-platform (mostly), and self-documenting. Let's dive in.

---

## Table of Contents

- [Resources](#resources)
- [Resource Types Reference](#resource-types-reference)
- [Variables](#variables)
- [Data Types](#data-types)
- [Strings and Interpolation](#strings-and-interpolation)
- [Arrays and Hashes](#arrays-and-hashes)
- [Conditionals](#conditionals)
- [Classes](#classes)
- [Defined Types](#defined-types)
- [Relationships and Ordering](#relationships-and-ordering)
- [Functions](#functions)
- [Templates](#templates)
- [Node Definitions](#node-definitions)
- [Iteration](#iteration)
- [Regular Expressions](#regular-expressions)

---

## Resources

Resources are the **fundamental building blocks** of Puppet. Every resource declaration follows this pattern:

```puppet
type { 'title':
  attribute1 => value1,
  attribute2 => value2,
}
```

- **Type**: What kind of thing (e.g., `file`, `package`, `service`, `user`)
- **Title**: A unique identifier within that type (must be unique per type in a catalog)
- **Attributes**: Key-value pairs describing the desired state

### Resource Titles vs. Namevar

Most resource types have a **namevar** — a special attribute that defaults to the title. For `file`, the namevar is `path`. For `package`, it's `name`. This means:

```puppet
# These two are equivalent:
file { '/etc/motd':
  content => 'Welcome!\n',
}

file { 'motd_file':
  path    => '/etc/motd',
  content => 'Welcome!\n',
}
```

The first form uses the title as the path (because `path` is the namevar for `file`). The second form uses a descriptive title and specifies the path explicitly. Both are valid — use whichever is clearer.

---

## Resource Types Reference

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

## Variables

Variables in Puppet start with `$` and are **immutable** once assigned (no reassignment within the same scope):

```puppet
# Simple assignment
$greeting = 'Hello, OpenVox!'
$port     = 8080
$enabled  = true

# Using variables
file { '/etc/motd':
  content => "${greeting}\nListening on port ${port}\n",
}
```

### Facts as Variables

System facts are available via the `$facts` hash:

```puppet
# Access facts using the $facts hash (preferred)
notify { "Running on ${facts['os']['name']} ${facts['os']['release']['major']}": }

# Check architecture
if $facts['os']['architecture'] == 'x86_64' {
  notify { 'This is a 64-bit system': }
}
```

### Variable Scope

Variables are scoped to the class or defined type they're declared in:

```puppet
class myapp {
  $config_dir = '/etc/myapp'  # Available within this class

  file { "${config_dir}/app.conf":
    ensure => file,
  }
}

# Access from outside: $myapp::config_dir
# (only if the class has been declared)
```

---

## Data Types

Puppet has a rich type system. Here are the types you'll use most:

| Type | Examples | Notes |
|------|----------|-------|
| `String` | `'hello'`, `"world ${var}"` | Single or double quotes |
| `Integer` | `42`, `-7`, `0xFF` | Whole numbers |
| `Float` | `3.14`, `-0.5` | Decimal numbers |
| `Boolean` | `true`, `false` | Note: lowercase! |
| `Array` | `[1, 'two', true]` | Ordered list |
| `Hash` | `{'key' => 'value'}` | Key-value pairs |
| `Undef` | `undef` | No value (like null) |
| `Regexp` | `/^web\d+/` | Regular expression |
| `Sensitive` | `Sensitive('s3cret')` | Redacted in logs |

### Type Validation in Parameters

```puppet
class myapp (
  String           $app_name,
  Integer[1, 65535] $port = 8080,
  Boolean          $debug = false,
  Array[String]    $allowed_hosts = ['localhost'],
  Optional[String] $custom_header = undef,
) {
  # Parameters are type-checked at compile time!
}
```

---

## Strings and Interpolation

Puppet has two kinds of strings:

```puppet
# Single-quoted: literal (no interpolation, no escape sequences except \\ and \')
$literal = 'The variable $name is not expanded here'

# Double-quoted: interpolated (variables and escape sequences work)
$name = 'OpenVox'
$interpolated = "Welcome to ${name}!\n"  # → "Welcome to OpenVox!\n"
```

### Heredocs

For multi-line strings, use heredocs:

```puppet
$config = @("CONFIG")
  # Application configuration
  server_name = ${facts['networking']['fqdn']}
  listen_port = ${port}
  log_level   = info
  | CONFIG

# The | strips leading whitespace up to the pipe character
```

---

## Arrays and Hashes

```puppet
# Arrays
$packages = ['httpd', 'php', 'mysql']
$first    = $packages[0]     # 'httpd'
$count    = length($packages) # 3

# Hashes
$user_config = {
  'name'  => 'deploy',
  'uid'   => 5000,
  'shell' => '/bin/bash',
}
$username = $user_config['name']  # 'deploy'

# Nested access
$os_name = $facts['os']['name']  # 'Rocky'
```

---

## Conditionals

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

## Classes

Classes are **named blocks of Puppet code** that can be declared (included) on a node. They're the primary way to organize your Puppet code.

### Defining a Class

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

### Declaring (Using) a Class

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

### The `contain` and `require` Functions

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

## Defined Types

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

## Relationships and Ordering

By default, Puppet applies resources in a **non-deterministic order**. To enforce ordering, use relationships:

### Arrow Notation

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

### Metaparameters

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

### Chaining

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

## Functions

Puppet includes many built-in functions:

```puppet
# String functions
$upper   = upcase('hello')           # 'HELLO'
$trimmed = strip('  spaces  ')       # 'spaces'
$joined  = join(['a', 'b', 'c'], ',')# 'a,b,c'

# Array functions
$unique  = unique([1, 2, 2, 3])      # [1, 2, 3]
$flat    = flatten([[1, 2], [3, 4]]) # [1, 2, 3, 4]
$sorted  = sort([3, 1, 2])          # [1, 2, 3]

# Hash functions
$merged = merge({'a' => 1}, {'b' => 2})  # {'a' => 1, 'b' => 2}

# Lookup (Hiera)
$db_host = lookup('myapp::db_host')
$db_port = lookup('myapp::db_port', Integer, 'first', 5432)

# Type checking
$is_str = $value =~ String   # true if $value is a String

# Conditional with assert
assert_type(String, $name) |$expected, $actual| {
  fail("Expected ${expected}, got ${actual}")
}
```

---

## Templates

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

## Node Definitions

Node definitions let you assign code to specific nodes in your `site.pp`:

```puppet
# manifests/site.pp

# Default for all nodes
node default {
  include base_profile
}

# Specific node
node 'webserver1.example.com' {
  include base_profile
  include webserver
  include monitoring
}

# Regex matching
node /^db\d+\.example\.com$/ {
  include base_profile
  include database
}
```

> **Pro tip:** In modern Puppet/OpenVox, many teams use **roles and profiles** with Hiera instead of node definitions. This is more flexible and easier to manage at scale. See the [Module Development Guide](../module-development/README.md) for the roles/profiles pattern.

---

## Iteration

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

## Regular Expressions

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

## Putting It All Together: A Real-World Example

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

---

*Next up: [CLI Reference →](../cli-reference/README.md)*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
