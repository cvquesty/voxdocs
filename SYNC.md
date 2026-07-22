# Keeping community VoxDocs factually aligned with official OpenVox docs

**This repository** (`cvquesty/voxdocs`) is a **community companion** to the official
OpenVox documentation. It has its own:

- Docsify shell and branding
- Section structure and navigation
- Conversational writing style

The official site is **canonical for product facts**:

**https://docs.openvoxproject.org/**  
Source: [OpenVoxProject/openvox-docs](https://github.com/OpenVoxProject/openvox-docs)

See also **[EDITORIAL.md](EDITORIAL.md)** for how we merge official facts with community color.

## What “sync” means here

| Do | Don’t |
|----|--------|
| Cross-check versions, platforms, package names, prerequisites | Overwrite this site with their HTML/VitePress design |
| Update **our** Markdown when official facts change | Mirror the entire official website into production |
| Keep lab CLI captures dated and honest | Pretend lab output is always “latest” |

## Production hosting

| Item | Value |
|------|--------|
| Site | https://voxdocs.questy.org |
| Host | server.questy.org |
| Docroot | `/var/www/html/voxdocs` |
| Deploy | rsync/git publish of **this** repo (Docsify + Markdown) |

## Weekly content audit (Saturday 02:00 America/New_York)

On the server:

| Piece | Location |
|-------|----------|
| Audit script | `/usr/local/sbin/voxdocs-content-audit.sh` |
| Timer | `voxdocs-content-audit.timer` |
| Report | `/var/log/voxdocs-content-audit.log` |

The audit **does not rewrite the live site**. It reports when GitHub latest tags for
OpenVox components drift from versions claimed in this repo’s `README.md` /
`AGENTS.md`, so a human (or a follow-up PR) can update **our** Markdown.

```bash
# Manual audit
sudo /usr/local/sbin/voxdocs-content-audit.sh

# Status
systemctl list-timers voxdocs-content-audit.timer
```

## Privacy

Never publish real lab FQDNs. Use **“a live lab”** or `*.example.com`.
