# Resources

[← Back to Language Reference](README.md)

---


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

Most resource types have a **namevar** — a special attribute that identifies the real-world thing being managed. For `file` the namevar is `path`, for `package` and `service` it's `name`, for `user` it's also `name`. When you don't set the namevar explicitly, **Puppet uses the title as its value**.

This gives you two ways to write any resource:

```puppet
# Form 1: Title IS the resource identity (shorthand)
# The title '/etc/motd' is automatically used as the 'path' parameter.
file { '/etc/motd':
  content => "Welcome!\n",
}

# Form 2: Descriptive title + explicit namevar
# When the title is NOT the resource identity, you MUST specify the namevar.
file { 'message_of_the_day':
  path    => '/etc/motd',
  content => "Welcome!\n",
}
```

Both declarations manage the exact same file — they are functionally identical. Form 1 is concise and works well when the resource identity is short and clear. Form 2 is useful when paths are long, when you want the title to describe intent, or when multiple resources might manage files in similar locations.

Here's the same pattern applied to other resource types:

```puppet
# Package: title as name (shorthand)
package { 'httpd':
  ensure => installed,
}

# Package: descriptive title + explicit name
package { 'web_server_package':
  name   => 'httpd',
  ensure => installed,
}

# Service: title as name (shorthand)
service { 'httpd':
  ensure => running,
  enable => true,
}

# Service: descriptive title + explicit name
service { 'web_server_service':
  name   => 'httpd',
  ensure => running,
  enable => true,
}
```

**Common namevars by type:**

| Resource Type | Namevar | What It Identifies |
|---------------|---------|-------------------|
| `file` | `path` | Absolute path on disk |
| `package` | `name` | Package name |
| `service` | `name` | Service name |
| `user` | `name` | Username |
| `group` | `name` | Group name |
| `cron` | `name` | Cron job identifier |
| `exec` | `command` | Command to execute |

> **Rule of thumb:** If the title looks like the actual thing (a path, a package name, a service name), you're using the shorthand and Puppet will figure it out. If the title is descriptive (like `'web_server_package'`), you *must* explicitly set the namevar parameter or Puppet won't know what to manage.

---

<sub>This document was created with the assistance of AI (Grok, xAI). All technical content has been reviewed and verified by human contributors.</sub>
