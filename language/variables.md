# Variables & Data Types

[← Back to Language Reference](README.md)

---


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

[← Back to Language Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
