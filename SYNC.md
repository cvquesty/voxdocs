# Keeping voxdocs.questy.org in sync with official OpenVox docs

Production **voxdocs.questy.org** is a **mirror** of the published site:

**https://docs.openvoxproject.org/**

Source project: [OpenVoxProject/openvox-docs](https://github.com/OpenVoxProject/openvox-docs)

## How automation works

On **server.questy.org**:

| Piece | Location |
|-------|----------|
| Sync script | `/usr/local/sbin/voxdocs-sync-openvox.sh` |
| systemd service | `voxdocs-sync.service` (oneshot) |
| systemd timer | `voxdocs-sync.timer` |
| Live web root | `/var/www/html/voxdocs` |
| Staging cache | `/var/cache/voxdocs-sync/` |
| Previous tree | `/var/www/html/voxdocs.prev` (last successful live) |

### Schedule

**Every Saturday at 02:00 America/New_York** (Eastern — “Saturday night / early morning”).

```bash
systemctl list-timers voxdocs-sync.timer
```

### Manual sync

```bash
sudo systemctl start voxdocs-sync.service
# or
sudo /usr/local/sbin/voxdocs-sync-openvox.sh
```

### What the script does

1. `wget` recursive mirror of `https://docs.openvoxproject.org/`
2. Sanity-check (`index.html`, `assets/`, minimum file count)
3. Preserve `/.well-known` for Let’s Encrypt
4. Atomic publish into `/var/www/html/voxdocs`
5. Keep previous tree as `voxdocs.prev` for quick rollback

```bash
# Rollback to previous publish
sudo rm -rf /var/www/html/voxdocs
sudo mv /var/www/html/voxdocs.prev /var/www/html/voxdocs
```

## This Git repository

The Markdown guides in **this** repo (`cvquesty/voxdocs`) are the older **community-written** documentation set. Production no longer serves them by default after the official mirror was enabled (2026-07-22).

Community content may still exist in an archive on the server under `/var/www/html/_archive/`.

## Privacy / hostname redaction

Every sync run **redacts private lab identifiers** from the mirrored tree before
publish, so the public site never advertises internal hosts:

| Original pattern | Replacement |
|------------------|-------------|
| `openvox.questy.org` (and similar private FQDNs in content) | `a live lab` |
| `agent1.questy.org` / `agent2.questy.org` | `agent1.example.com` / `agent2.example.com` |
| `10.0.100.x` lab IPs | `192.168.1.x` examples |

Community Markdown in this repository should also use **“a live lab”** or
`*.example.com` — never real production/lab FQDNs.

