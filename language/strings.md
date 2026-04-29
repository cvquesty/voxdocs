# Strings, Arrays & Hashes

[← Back to Language Reference](README.md)

---

Puppet has two kinds of strings:

```puppet
# Single-quoted: literal (no interpolation, no escape sequences except \\ and \')
$literal = 'The variable $name is not expanded here'

# Double-quoted: interpolated (variables and escape sequences work)
$name = 'OpenVox'
$interpolated = "Welcome to ${name}!\n"  # → "Welcome to OpenVox!\n"
```

## Heredocs

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

[← Back to Language Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
