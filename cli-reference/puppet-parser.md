# puppet parser

> *Your syntax-checking safety net — run it before every commit.*

[← Back to CLI Reference](README.md)

---

Validate and interact with Puppet manifests.

```
$ puppet help parser

USAGE: puppet parser <action>

Interact directly with the parser.

OPTIONS:
  --render-as FORMAT             - The rendering format to use.
  --verbose                      - Whether to log verbosely.
  --debug                        - Whether to log debug information.

ACTIONS:
  dump        Outputs a dump of the internal parse tree for debugging
  validate    Validate the syntax of one or more Puppet manifests.
```

## Common Usage Patterns

```bash
# Validate a manifest (check for syntax errors)
puppet parser validate manifest.pp

# Validate all .pp files in a directory
find . -name '*.pp' -exec puppet parser validate {} +

# Dump the AST (Abstract Syntax Tree)
puppet parser dump manifest.pp
```

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>