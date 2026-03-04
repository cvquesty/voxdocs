# Functions

[← Back to Language Reference](README.md)

---


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

[← Back to Language Reference](README.md)

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
