# Functions

[← Back to Language Reference](README.md)

---

Puppet includes many built-in functions for string manipulation, data transformation, lookups, and more.

---

## String Functions

```puppet
# Case transformation
$upper = upcase('hello')              # 'HELLO'
$lower = downcase('HELLO')            # 'hello'
$cap   = capitalize('hello world')    # 'Hello world'
$camel = camelcase('my_variable')     # 'MyVariable'

# Whitespace handling
$trimmed = strip('  spaces  ')        # 'spaces'
$left    = lstrip('  left')           # 'left'
$right   = rstrip('right  ')          # 'right'
$chomped = chomp("line\n")            # 'line'

# String operations
$joined  = join(['a', 'b', 'c'], ',') # 'a,b,c'
$split   = split('a,b,c', ',')        # ['a', 'b', 'c']
$length  = length('hello')            # 5

# Search and replace
$replaced = regsubst('hello world', 'world', 'OpenVox')  # 'hello OpenVox'
$matched  = 'web01' =~ /^web/         # true
```

---

## Array Functions

```puppet
$unique    = unique([1, 2, 2, 3])         # [1, 2, 3]
$flat      = flatten([[1, 2], [3, 4]])    # [1, 2, 3, 4]
$sorted    = sort([3, 1, 2])              # [1, 2, 3]
$reversed  = reverse([1, 2, 3])           # [3, 2, 1]
$size      = size(['a', 'b', 'c'])        # 3
$empty     = empty([])                    # true
$first     = ['a', 'b', 'c'][0]           # 'a'
$last      = ['a', 'b', 'c'][-1]          # 'c'
$sliced    = ['a', 'b', 'c', 'd'][1, 2]   # ['b', 'c']
```

---

## Hash Functions

```puppet
$merged = merge({'a' => 1}, {'b' => 2})   # {'a' => 1, 'b' => 2}
$keys   = keys({'a' => 1, 'b' => 2})      # ['a', 'b']
$values = values({'a' => 1, 'b' => 2})    # [1, 2]
$has    = 'a' in {'a' => 1, 'b' => 2}     # true

# Deep access
$nested = {'db' => {'host' => 'localhost'}}
$host   = $nested['db']['host']           # 'localhost'
$safe   = $nested.dig('db', 'port')       # undef (no error if missing)
```

---

## Hiera Lookup (The Modern Way)

The `lookup()` function is the **standard way to retrieve data from Hiera** in Puppet 8 / OpenVox 8:

```puppet
# Basic lookup
$db_host = lookup('myapp::db_host')

# With type validation
$db_port = lookup('myapp::db_port', Integer)

# With default value
$timeout = lookup('myapp::timeout', Integer, 'first', 30)

# With merge strategy
$packages = lookup('profile::base::packages', Array[String], 'unique')

# Hash merge
$config = lookup('myapp::config', Hash, 'deep')
```

### Lookup Signatures

| Form | Description |
|------|-------------|
| `lookup('key')` | Simple lookup, returns first match |
| `lookup('key', Type)` | With type validation |
| `lookup('key', Type, 'strategy')` | With merge strategy |
| `lookup('key', Type, 'strategy', default)` | With default if not found |

### Merge Strategies

| Strategy | Behavior |
|----------|----------|
| `'first'` | Return first match (default) |
| `'unique'` | Merge arrays, remove duplicates |
| `'hash'` | Shallow merge of hashes |
| `'deep'` | Recursive deep merge of hashes |

> **⚠️ Deprecated:** The legacy `hiera()`, `hiera_array()`, `hiera_hash()`, and `hiera_include()` functions are **removed in Puppet 8 / OpenVox 8**. Always use `lookup()` instead. See [Migrating from Puppet 7](../getting-started/migration.md) for details.

---

## Type Checking

```puppet
# Type matching with =~
$is_string  = $value =~ String            # true if String
$is_integer = $value =~ Integer           # true if Integer
$is_array   = $value =~ Array             # true if Array

# Type assertion (fails compilation if wrong type)
assert_type(String, $name) |$expected, $actual| {
  fail("Expected ${expected}, got ${actual}")
}

# Safe type conversion
$as_string = String($port)                # Convert to string
$as_int    = Integer('42')                # Convert to integer
```

---

## Conditional Functions

```puppet
# Return value based on condition
$result = if $facts['os']['family'] == 'RedHat' { 'yum' } else { 'apt' }

# Pick first defined value
$port = pick($custom_port, $default_port, 8080)

# Pick first defined, non-empty value
$name = pick_default($user_name, 'anonymous')

# Conditional with .lest (if undef, run block)
$value = lookup('optional::key', undef, undef, undef).lest || { 'default' }

# Conditional with .then (if defined, transform)
$upper_name = $name.then |$n| { upcase($n) }
```

---

## Useful Utility Functions

```puppet
# Generate content from template
$config = epp('mymodule/config.epp', { 'port' => 8080 })

# Include a class
include mymodule::install

# Contain a class (with relationship inheritance)
contain mymodule::service

# Fail with error message
if $port < 1 or $port > 65535 {
  fail("Invalid port: ${port}")
}

# Debug output (only in debug mode)
debug("Processing node ${facts['networking']['fqdn']}")

# Notification levels
notice('This is informational')
warning('This might be a problem')
err('This is an error')
```

---

## See Also

- [Hiera Deep-Dive](../hiera/README.md) — Complete guide to `lookup()` and data hierarchies
- [Iteration](iteration.md) — `each`, `map`, `filter`, `reduce`
- [Templates](templates.md) — EPP and ERB template functions

---

[← Back to Language Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
