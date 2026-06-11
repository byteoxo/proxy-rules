# Agent guide

Instructions for AI agents and contributors editing **proxy-rules**.

## Project purpose

This repository stores **classical proxy rule snippets** (domain / IP / process matchers) as plain `.txt` files. It does **not** ship full Clash YAML, Surge `.conf`, or proxy server definitions. Consumers attach each file to a policy in their own client config.

Primary audience: developers in China who need reliable proxy routing for foreign dev tooling, AI services, and selected apps — plus `DIRECT` lists for domestic traffic.

## Directory conventions

```
proxy-rules/
├── *.txt                 # Top-level topic rules (openai, cursor, ai, …)
├── development/          # Dev registries & tooling (umbrella + per-ecosystem slices)
├── cn/                   # China domestic — intended for DIRECT policy
├── social-media/
└── crypto/
```

### When to use which file

| Situation | Action |
|-----------|--------|
| New foreign SaaS / IDE / API (single product) | Add or extend a **top-level** `servicename.txt` |
| Package registry, CDN, or language ecosystem | Add to the matching `development/<ecosystem>.txt` slice **and** consider `development/development.txt` if broadly useful |
| Domestic China service that must not be proxied | Add under `cn/` |
| Social or crypto wallet domain | Add under `social-media/` or `crypto/` |

Avoid creating deeply nested folders. Keep one concern per file.

### Umbrella vs slices

- `development/development.txt` is the **aggregated** dev list. When adding domains that belong in an existing slice (`node.txt`, `rust.txt`, …), update the slice and mirror the entry into `development.txt` if it is generally useful.
- Slices (`github.txt`, `node.txt`, …) exist so users can import minimal rule sets. Do not delete slices in favor of only the umbrella file.

## Rule syntax

Supported matchers (Clash / Surge / Stash classical):

```
DOMAIN-SUFFIX,example.com
DOMAIN,host.example.com
DOMAIN-KEYWORD,fragment
PROCESS-NAME,/Applications/App.app/
IP-CIDR,x.x.x.x/32,no-resolve
IP-CIDR6,xxxx::/32,no-resolve
IP-ASN,nnnnn,no-resolve
USER-AGENT,AppName*
```

### Do

- Use `DOMAIN-SUFFIX` when the entire registrable domain should match.
- Use `DOMAIN` for exact hostnames or CDN edge names that should not broaden to sibling subdomains.
- Use `DOMAIN-KEYWORD` sparingly — it is broad and can cause false positives.
- Append `,no-resolve` to all `IP-*` rules.
- Add a short `#` header comment when a file's purpose is not obvious from the filename.
- Preserve attribution headers when importing from external rule sets (see `cn/wechat.txt`).

### Do not

- Append policy names (`PROXY`, `DIRECT`, `REJECT`) to rule lines — consumers assign policy in their config.
- Add duplicate lines within the same file.
- Widen rules unnecessarily (e.g. `DOMAIN-SUFFIX,com` or `DOMAIN-SUFFIX,microsoft.com` when only one subdomain is needed).
- Commit secrets, personal domains, or one-off LAN addresses.
- Modify large third-party lists (WeChat, 12306) unless syncing from upstream — prefer updating attribution date and source URL in the header.

## File naming

- Lowercase, descriptive, underscore-separated: `arc_browser.txt`, `walletconnect.txt`.
- Top-level files: product or region name (`openai.txt`, `russia.txt`).
- `development/` slices: ecosystem name (`node.txt`, `java.txt`).

## Editing checklist

Before finishing a change:

1. **Correct bucket** — foreign/proxy candidate vs `cn/` direct list.
2. **No duplicates** — grep the repo for the domain if unsure.
3. **Minimal scope** — smallest matcher that reliably covers the service.
4. **Comments** — section headers in large files (`# --- Node / npm ---`).
5. **Sync umbrella** — if editing a `development/` slice, update `development.txt` when appropriate.

## Commit messages

Follow existing style:

```
feat: add domains for <service>
feat: update rules
fix: remove duplicate githubusercontent entry
```

Keep commits focused on rule changes only.

## Testing

There is no automated test suite. Validate informally:

- Lines are non-empty, comma-separated, valid matcher prefix.
- No trailing whitespace on rules.
- Imported lists keep valid syntax (especially `no-resolve` on IP rules).

If the user runs Clash Meta locally, suggest a dry run: load the file as a `rule-provider` and confirm the config parses.

## Common integrations (for user docs)

**Clash rule-provider (HTTP):**

```yaml
rule-providers:
  example:
    type: http
    behavior: classical
    url: https://raw.githubusercontent.com/byteoxo/proxy-rules/main/openai.txt
    path: ./ruleset/openai.txt
    interval: 86400
```

**Surge:**

```ini
RULE-SET,https://raw.githubusercontent.com/byteoxo/proxy-rules/main/openai.txt,PROXY
```

Remind users: `cn/*` sets usually belong **before** broad `PROXY` rules with policy `DIRECT`.

## Out of scope

Do not add to this repo:

- Full proxy client configurations
- GeoIP databases or ASN DB files
- Scripts that merge rules (unless explicitly requested)
- Non-classical rule formats (unless the project later standardizes on them)

## Questions to ask the user

When requirements are ambiguous:

1. Should the domain be **proxied** or **direct**?
2. Is this a one-off hostname or an entire `DOMAIN-SUFFIX`?
3. Should it go in a new top-level file or an existing `development/` slice?

Default: foreign developer/AI tooling → proxy-oriented top-level or `development/` file; Chinese domestic consumer service → `cn/` with `DIRECT` pairing.
