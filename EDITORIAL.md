# Editorial policy — community VoxDocs × official OpenVox docs

## Roles

| Source | Role |
|--------|------|
| **[docs.openvoxproject.org](https://docs.openvoxproject.org/)** | **Canonical for product facts** — component versions, install methods, platform support, config keys, architecture terminology, security model, prerequisites |
| **This repo (`cvquesty/voxdocs`)** | **Community companion** — our Docsify design, section map, humor, lab CLI captures, practical “how we run it” color, and topics official docs don’t cover as deeply |

## When both cover a topic

1. **Facts** come from official docs (or OpenVoxProject release tags when docs lag by a day).
2. **Narrative, jokes, diagrams, lab paste, and opinion** stay ours unless they contradict a fact.
3. The result is **one continuous guide** — not two competing copies, not a wholesale paste of official HTML.
4. Prefer **OpenVox product names** in prose (`OpenVox agent`, `OpenVox server`, `OpenVoxDB`) while documenting **on-disk compatibility names** (`puppet`, `puppetserver`, `puppetdb` units and paths).

## When only we have material

Keep it. If official docs are silent, our content stands. Label lab-specific captures with a date.

## When only they have material

Summarize the fact in our voice and **link** to the official page for depth. Don’t rewrite our site into their book.

## Hostnames

Use **a live lab** / `*.example.com` — never real lab FQDNs.

## Automation

Weekly **content audit** only (versions). Full literary merge is human/editor work guided by this file.
