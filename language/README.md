# 📝 The Puppet Language Reference

> *"It's not programming, it's declaring your intentions to the universe. The universe just happens to be made of servers."*

---

> **Sources:** Language semantics follow official OpenVox language docs
> ([lang summary](https://docs.openvoxproject.org/openvox/latest/lang_summary.html) and related pages).
> Examples, analogies, and lab-flavored samples are ours — [EDITORIAL.md](../EDITORIAL.md).

---

## Overview

The Puppet language (also called the Puppet DSL — OpenVox’s configuration language) is a **declarative, domain-specific language** designed for one thing: describing the desired state of your infrastructure. You don't tell Puppet *how* to do something — you tell it *what you want*, and it figures out the rest.

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

Each topic has its own page with detailed explanations and copy-pasteable examples.

### Language Basics

| # | Topic | Description |
|---|-------|-------------|
| 1 | [**Resources**](resources.md) | The fundamental building blocks — types, titles, and namevars |
| 2 | [**Resource Types Reference**](resource-types.md) | file, package, service, user, group, cron, exec — with examples |
| 3 | [**Variables & Data Types**](variables.md) | Variables, facts, scope, type system, and parameter validation |
| 4 | [**Strings, Arrays & Hashes**](strings.md) | String interpolation, heredocs, arrays, hashes, and nested access |

### Control Flow & Structure

| # | Topic | Description |
|---|-------|-------------|
| 5 | [**Conditionals**](conditionals.md) | if/elsif/else, case statements, and selectors |
| 6 | [**Classes**](classes.md) | Defining, declaring, and parameterizing classes |
| 7 | [**Defined Types**](defined-types.md) | Reusable resource templates — instantiate multiple times |
| 8 | [**Relationships & Ordering**](relationships.md) | Arrow notation, metaparameters, and chaining |
| 9 | [**Iteration**](iteration.md) | each, map, filter, and reduce |

### Advanced Topics

| # | Topic | Description |
|---|-------|-------------|
| 10 | [**Functions**](functions.md) | Built-in functions for strings, arrays, hashes, and lookups |
| 11 | [**Templates**](templates.md) | EPP (preferred) and ERB templates for dynamic file content |
| 12 | [**Node Definitions**](node-definitions.md) | Assigning code to specific nodes in site.pp |
| 13 | [**Regular Expressions**](regex.md) | Ruby-style regex matching, capture groups, and case statements |

### Putting It Together

| # | Topic | Description |
|---|-------|-------------|
| 14 | [**Real-World Example**](example.md) | A complete NTP module using everything you've learned |

---

*Next up: [CLI Reference →](../cli-reference/README.md)*

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
